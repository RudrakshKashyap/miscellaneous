# SLO Alert Registry

Working sheet for deciding which flexipill-core flows get alerts, at what threshold, and who answers them.

Two sheets, same 17 columns and same method:

| File | Scope | Owner |
|---|---|---|
| `alert-registry.csv` | Platform, infra, orders, zoho/SO | shared / TBD |
| `fno-team-alerts.csv` | **transfer-order** — critical flows only | FNO |
| `polaris-alerts.csv` | **Polaris** (CS tool) — agent loop + FE | TBD |
| `fulfilment-alerts.csv` | **warehouse ops** — inward (gatepass, GRN) + outward (picker, invoice, manifest) | TBD |

Open in Numbers/Excel. One flow gets **two rows** — `critical` (page a human) and
`warning` (Slack, business hours).

## FNO sheet — transfer-order

Scoped deliberately to the five flows where a failure loses stock or money, not every
endpoint. The module has 18 routes; most of them failing is a bad afternoon, not an
incident.

| Flow | Why it's critical |
|---|---|
| Stuck reservation | Stock stops being sellable with no error anywhere — the module's defining failure |
| Zoho draft failed | **API returns 201** while the TO holds stock and cannot be picked. No HTTP alert can see this |
| Send-to-transit | Point of no return, spans two Zoho round trips, partial completion leaves split state |
| Reconcile cron | The only self-healing mechanism — if it's dead, the other four degrade silently with it |
| Picker starvation | Eligibility is a Redis-only gate that fails **closed** and silently |

**Do the reconcile/stuck-reservation cron check first.** Neither
`POST /transfer-orders/cron/reconcile` nor `POST /transfer-orders/cron/stuck-reservations`
appears in the cron inventory. `TRANSFER_ORDER_STUCK_RESERVATION_DETECTED` is only ever
emitted by that second endpoint — so if it isn't scheduled, the 48-hour stuck-reservation
alarm the code was designed around does not exist, and the top row of the sheet can never
fire. Verify on cron-job.org before building anything else.

One code change worth making while you're here: log a line when the parsed picker
eligibility list is empty. That single log replaces the awkward composite alarm in
`to-picker-starved-crit` with a direct signal.

## Method

Work **module by module**, and within a module **flow by flow** — not endpoint by endpoint. An "API" is the wrong unit here because a single business outcome (an order getting a Sales Order) is produced by a queue consumer, a cron sweeper, and an inline stage path, and can fail in five distinct ways that need five different responses.

For each flow, ask in this order:

1. **What breaks for a customer or for ops if this fails?** If the honest answer is "nothing anyone notices", there is no alert here.
2. **What is the business SLI?** Prefer a signal that measures the *outcome* (orders with no SO) over one that measures a *cause* (Zoho threw an error). Outcome signals catch failure modes you did not predict.
3. **What is the noise floor?** Almost every TraceCode has one. `INSUFFICIENT_QTY` is a normal business event. `CREATE_CONTACT_AND_ZOHO_SALES_ORDER_ERROR` fires benignly on the Zoho 110802 tax retry. Write the floor into `notes` before picking a number.
4. **Ratio or count?** If the signal scales with traffic, it must be a ratio with a `min_volume_guard`. A count-based alarm on a traffic-scaled signal pages at 3am on one bad request out of eight.
5. **Critical vs warning.** Critical = someone must act now, out of hours. Warning = it can wait until morning. If you cannot write a runbook line for the critical row, it is not critical.

## Columns

| Column | Meaning |
|---|---|
| `module` | Source module the flow lives in |
| `flow` | The business flow, not the endpoint |
| `alert_id` | Unique slug; becomes the CloudWatch alarm name |
| `severity` | `critical` (page) or `warning` (Slack) |
| `what_breaks` | Plain-language impact. Written for whoever gets woken up |
| `signal` | TraceCode name, or the ratio / query |
| `signal_type` | `log-tracecode` \| `log-metric-value` \| `log-ratio` \| `db-query` |
| `cloudwatch_filter_pattern` | Exact metric-filter pattern (see below) |
| `metric_name` | CloudWatch custom metric name the filter writes to |
| `threshold` | Firing condition |
| `window` | Evaluation period |
| `min_volume_guard` | Minimum denominator; mandatory on every ratio |
| `action` | `Page` \| `Slack` \| `Ticket` |
| `owner` | Who answers it |
| `runbook` | One line: first thing to check |
| `notes` | Noise floor, known false positives, caveats |
| `status` | `proposed` → `baselining` → `live` |

## CloudWatch specifics for this repo

**The log JSON is double-wrapped.** `LoggingService` writes through a Winston printf that emits:

```json
{ "level": "error", "msg": "<TRACECODE_NAME>", "meta": { ...everything else... } }
```

([logging.service.ts:27-29](../../flexipill-core/src/logging-service/logging.service.ts#L27-L29))

So metric filters match on **`$.msg`**, not `$.message`, and every context field is under **`$.meta.*`**:

```
{ $.msg = "SALES_ORDER_PENDING_ORDERS" }        →  metricValue: $.meta.pendingOrdersCount
{ $.msg = "CREATE_CONTACT_AND_ZOHO_SALES_ORDER_ERROR" }
```

Getting this wrong produces a filter that silently matches nothing — always confirm with Test Pattern against real log events before creating the alarm.

Other notes:

- JSON logging only happens when `awsMonitoringEnabled || datadog.enabled` **and** `NODE_ENV` is not `local`/`test`. Stage and prod are fine; local emits Nest's non-JSON format.
- Log group: `platinumrx-core-logs` (ap-south-1).
- SO creation runs on the AmazonMQ consumer, gated by `ENABLE_SINGLE_MESSAGE_PROCESSOR_QUEUE_CONSUMER`. That env var is **not** in any deployment yaml in the repo — it comes from a ConfigMap. Confirm which of the five deployments carries it before scoping any alarm to a log stream.
- CloudWatch has no native multi-window alarm. For burn-rate style alerts, build two alarms and AND them with a **composite alarm**.
- Max alarm period is 1 day, so any window longer than that needs `datapointsToAlarm` rather than a native window.

## Fulfilment sheet — warehouse ops, two chains

The sheet covers both directions of warehouse work. They're separate chains with
separate owners; the `module` column splits them.

**Inward** — stock arriving:
```
gatepass ─► GRN ─► Zoho bill ─► RACKING ─► findable stock ─► (Zoho-pull cron) ─► sellable
```

**Outward** — orders leaving:
```
SO created ─► picking ─► QC ─► INVOICE (55) ─► shipment (61) ─► manifest ─► dispatch
```

**Reverse (RTO)** — parcels coming back:
```
RTO_INITIATED (107) ─► IN_TRANSIT (108) ─► DELIVERED (112) ─► RECEIVED_AT_WAREHOUSE (113)
   ─► QUALITY_CHECK_DONE (125) ─► REFUND_INITIATED (160) ─► REFUND_PROCESSED (161)
terminal failures: RTO_LOST (117), RTO_MISMATCH (120)
```

Each stage gates the next, so a failure anywhere presents downstream as "orders piling
up" with no error of its own. **Always triage upstream-first.** Picking is gated on
`zohoOrderCreationStatus == SALES_ORDER_CREATED`, so an SO outage starves the pick queue
*without producing a single picker error* — check the zoho sheet before believing a
picker alert.

**Invoice is the chokepoint.** Shipment is gated on it, so `inv-create-error-crit` stalls
everything behind it. Keep `INVOICE_BLOCKED_SALES_ORDER_PENDING_EDIT` and
`INVOICE_BLOCKED_SALES_ORDER_NOT_READY` out of that metric — they're designed gates, not
failures.

Two things the code already does that the alerts lean on:

- **`CREATE_INVOICE_STEP_TIMING` emits a `phase: 'start'` marker before any work**
  (`orders.service.ts:20046-20054`) specifically so a hang that never reaches the summary
  still leaves a record. Started-minus-completed is therefore a **designed-in hang
  detector** — an invoice that starts and never finishes produces no error and no
  error-rate alert can see it. It also emits per-step durations, which is the only real
  step-level latency in the codebase.
- **`INVOICE_ASYNC_TASK_LAST_ATTEMPT_WARNING` fires before `INVOICE_QUEUE_MESSAGE_DROPPED`.**
  Treat the warning as the actionable one. Note `INVOICE_QUEUE_MAX_RECEIVE_COUNT` is only
  **2** (`invoice-async-task.constants.ts:51`), so tasks are abandoned after two attempts
  — far less forgiving than typical SQS defaults.

`man-dispatch-error` is the one alert here with a hard time dependency: it fires during
the courier pickup window, and a missed window costs a full day of transit. Worth paging
at lower volume during pickup hours than you would otherwise.

### Inward-specific notes

**Gatepass failures compound.** An open gatepass blocks the next one for the same
warehouse + vendor + bill (`DUPLICATE_ACTIVE_GATEPASS`), so a stuck status update doesn't
stay flat — each unclosed gatepass blocks that vendor's next delivery. Triage
gatepass-first when GRN errors appear: `GRN_GATEPASS_NUMBER_REQUIRED` and
`GRN_ALREADY_HAS_GATEPASS` are gatepass problems wearing a GRN error code.

**`grn-bill-wedged-crit` fires on a single occurrence, deliberately.**
`SEARCH_ZOHO_BILL_ERROR` means the bill POST timed out *and* the recovery search also
failed. The code comment states the consequence directly
(`zoho.service.ts:5973-5978`): that orphans the bill on Zoho with nothing persisted
locally and *permanently wedges the GRN*. Don't retry blind — check Zoho for the bill
first or you'll create a duplicate. Its healthy counterpart `CREATE_ZOHO_BILL_SKIPPED`
(the adopt-existing path) is tracked as a **warning** instead: a rising rate there means
Zoho bill POSTs are timing out routinely, which is the leading indicator that the wedged
case is coming.

**GST derivation failures aren't separately visible.** `GRN_GST_DERIVATION_ERROR` is
declared in the enum but **never emitted**, so a missing warehouse/vendor GSTIN is
indistinguishable from any other bill failure. Different fix (add the GSTIN vs. wait out
a Zoho outage), same signal. Worth emitting.

**`grn-funnel-crit` uses a 1-day window, not minutes.** A GRN spans a shift — staff start
inwarding, hit a wall partway, and abandon without any error being raised. It's the
inward equivalent of the invoice hang detector, but do not copy the short windows used
elsewhere in these sheets.

**A completed GRN is not sellable stock.** `item_procurement.stock_in_warehouse` is
updated by a Zoho-pull cron, *not* synchronously by `createBill` (repo `CLAUDE.md`
anti-patterns). So there's real lag between `GRN_COMPLETED` and the stock being
available. If the business SLO you actually want is "received stock becomes sellable
within N hours", that needs a separate signal further downstream — none of these rows
measure it.

**Racking sits between GRN and sellable stock**, and it's the step that turns received
stock into *findable* stock. A GRN can complete cleanly and the stock still be unpickable
if racking fails — it's sellable on paper and missing on the shelf. `rack-lookup-error`
degrades throughput rather than stopping it, so it surfaces as "picking is slow today"
rather than as an outage; it's paged anyway because the cost compounds across every order
in the shift.

**No racking funnel is buildable.** There is no `RACKING_TASK_CREATED` code, and both
`RACKING_TASK_CREATE_ERROR` and `ADD_MISSING_RACK_LOCATION_ERROR` are declared-only and
never emitted. So the created-vs-completed detector that works for GRN and invoices has
no signal here.

### Reverse chain (RTO) notes

**The RTO cron aborts on first failure.** `processRtoStatusUpdate` wraps the whole sweep
in one try/catch and rethrows, so a single bad order stops every order behind it — unlike
the transfer-order reconcile cron, which is per-item. That's why
`rto-cron-error-warn` fires on a single occurrence. The cron itself *is* scheduled
(every 2h, `/orders/updateRtoStatus`) — confirmed in the cron inventory, unlike the
transfer-order jobs.

**It also logs no count.** The handler emits the `PROCESS_RTO_STATUS_UPDATE_REQUEST`
marker with no payload and returns `{count}` without logging it. Adding `count` to that
log turns a bare liveness check into a real throughput signal — cheap, worth doing.

**Refund is gated on QC.** `RTO_REFUND_INITIATED` (160) can't happen until
`RTO_QUALITY_CHECK_DONE` (125), so a QC backlog presents as stuck refunds. Triage QC
first.

**The RTO rows need a signal decision before building.** Statuses 160/117/120/112 are
order-status transitions, and it isn't confirmed whether `UPDATE_ORDER_STATUS` carries
the status id in `$.meta`. If it does, these are metric filters; if not, they're Metabase
alerts on `marketplace.orders`. Check before writing the filters — the rows are marked
`VERIFY` for this reason.

**The quality gates are not an engineering alert.** `GRN_EXPIRY_BLOCK`,
`GRN_PTR_MRP_BLOCK` and `GRN_MRP_DEVIATION` are working-as-intended rejections; a spike
means a vendor problem. Routed to procurement as a warning only, with no critical row.

## Polaris sheet — and the frontend problem

Polaris is the customer-support workspace (`dashboard/src/pages/Polaris/Workspace`). The
whole tool is one loop:

```
Go Active   → PATCH /internal-users/status   → UPDATE_AGENT_STATUS_ERROR
Next task   → GET  /tasks/next               → SERVE_NEXT_TASK_SERVED | _EMPTY | _ERROR   ★
Load 360    → user details / orders / wallet   (Promise.allSettled — partial failure renders anyway)
Complete    → POST /tasks/disposition        → CREATE_DISPOSITION_ERROR
Follow-up   → POST /tasks/new                → CREATE_TASK_ERROR
```

`SERVE_NEXT_TASK_SERVED / _EMPTY / _ERROR` is the best-shaped SLI in the codebase — a
three-way served / empty / failed split, already emitted. Error-ratio gives you "agents
can't work"; empty-ratio gives you "nobody is creating tasks", which is the concern the
TL actually raised. Note `CRMTraceCodes` is a **string** enum on purpose (numeric values
used to reverse-map through `TraceCodes[code]` and log an unrelated name), so `$.msg`
carries these directly.

**There is no frontend telemetry.** No Sentry, no RUM, no ErrorBoundary anywhere in the
dashboard, no `window.onerror`, no `unhandledrejection` handler. A React render error
blanks the workspace and produces zero signal — two rows are filed
`blocked-needs-code-fix` for it.

How much this matters is smaller than it sounds, because Polaris is API-driven: most of
"the page isn't opening" is one of the calls above failing, and those are all
BE-visible. What stays invisible without a beacon:

- a render crash after successful API responses (white screen)
- bundle/asset load failure after a bad deploy
- token-expiry redirect loops — the interceptor bounces to `/`, which agents experience as "it logged me out again"

**The minimum viable fix is small, and the scaffolding already exists.**
`src/error-logging/error-logging.controller.ts` is an **empty stub** —
`@Controller('error-logging')` with no routes. Someone intended this and never built it.

1. Add a POST route there that logs a TraceCode with the posted payload.
2. FE: an ErrorBoundary plus `window.onerror` / `unhandledrejection` beaconing to it.
3. The axios response interceptor (`createAxios.js:95`) already catches every API error
   and currently only `console.log`s it — one line there covers API failures too.

After that, FE errors are ordinary CloudWatch log lines and everything else in this
README applies to them unchanged. Worth noting `getCustomHeaders()` already sends the FE
page `url` on every API call, so page attribution is already on the wire — it just isn't
logged server-side.

**One trap specific to this tool:** `useCustomer360` uses `Promise.allSettled` and renders
whatever resolved, tracking failures in a `failed{}` map that is used for rendering and
**never reported**. So a workspace with a dead wallet or orders widget looks like a
successful page load. "Page opened" ≠ "page usable" — `pol-customer360-crit` covers it
from the BE side, and it's the strongest argument for building the beacon.

## Catching our own bugs (not just third-party failures)

Integration alerts (Zoho down, broker dead) are the easy half. The harder half is
"someone shipped a regression". Three layers, in order of how much they catch:

**1. The generic 5xx net — currently broken.** The global exception filter
([exception-filter.ts](../../flexipill-core/src/common/middleware/exception-filter.ts))
catches every uncaught `Error`, returns a 500, and **logs nothing** — the `logError`
call is commented out. So any handler that throws outside its own try/catch fails
completely silently. This is exactly the "junior dev broke an API" case, and there is
no signal for it today. Two rows in the sheet are marked `blocked-needs-code-fix`
because of it.

Same filter, second problem: `QueryFailedError` is returned as **HTTP 400** with the
raw DB message. A broken query therefore looks like a *client* error in any HTTP-level
metric, and leaks schema detail to the caller.

Fix the filter before building anything else in this category. Everything below is a
partial substitute.

**2. Caught-error rate by route.** `logError` maps HTTP ≥500 → `$.level = "error"` and
400-499 → `$.level = "warn"`. `logInfo` stamps `$.meta.path` = `"<METHOD> <url>"` from
the AsyncLocalStorage request context (applied globally in `main.ts`). So per-route
error metrics are buildable from logs today:

```
{ $.level = "error" && $.meta.path = "POST /orders/initiateOrder*" }
```

Note `$.meta.path` is `req.url` — the **raw URL with query string and path params
substituted**, not the Nest route template. `GET /orders/12345` and `GET /orders/12346`
are different strings, so parameterised routes need a wildcard. Always confirm the
exact string with Test Pattern.

**2b. App-wide vs per-route — they catch different things.** The app has **837 routes
across 72 controllers**, so an app-wide error-rate alarm is a *catastrophe* detector only:
a route carrying 1% of traffic going 100%-broken moves the global rate by ~1 point, which
sits under any threshold worth setting. Per-route alarms are what catch one broken API.

Per-route doesn't scale by hand, though — and CloudWatch's default quota is 100 metric
filters per log group, so one-filter-per-route is structurally impossible, not just
tedious.

The scalable version is *one* metric filter **dimensioned** by route (metric filters
support up to 3 dimensions from log fields). **That is blocked today** because
`$.meta.path` is `req.url` — path params substituted, query string included — so its
cardinality is unbounded and every distinct URL would become a separately-billed custom
metric.

**Fix: add a global interceptor** (there is none today — the only `NestInterceptor` in the
codebase is a caching mixin) that stamps the *route template* and response status:

```
route: "POST /orders/:pg/callback"    // bounded: 837 values, not millions
statusCode: 500
```

One dimensioned filter on `route` then covers every endpoint automatically. The same
interceptor is the natural place to emit request duration, which does not exist anywhere
today (see Latency below). This is the highest-leverage change on the fix list.

Interim without code: Contributor Insights on `$.meta.path` (it is built for
high-cardinality fields, unlike metric filters) for triage, plus hand-picked filters on
the handful of routes that matter — which is what the `ord-*` rows do.

**3. Volume-drop canaries.** The failure neither of the above can see: an endpoint that
returns 200 and silently does nothing. Error rate reads 0%. The only detector is
"success events stopped happening" — `ord-volume-drop-crit` on `ORDER_CREATED`. Build
one of these for every flow where a silent no-op would be expensive. Compare against
the **same hour of day** over a trailing week, never a flat number.

## Infrastructure and dependency failures

The `infra` rows cover "the API is erroring for a reason that isn't our code" — DB down,
Redis down, pods dying, queues unconsumed. Three things worth knowing before you build
them.

**DB failures are the one thing that IS well instrumented.** `CustomTypeOrmLogger.logQueryError`
fires `DB_QUERY_ERROR` for **every** failed query, unsampled, and the logger is wired into
the datasource ([data-source.ts:37](../../platinumRx/flexipill-core/db/data-source.ts#L37)).
Unlike HTTP 5xx, you can actually see this today. `$.meta.query` tells you whether it's one
bad query (a deploy) or all of them (the database).

**A failing readiness probe logs nothing.** `checkHealth` throws
`InternalServerErrorException` ([pdp.service.ts:448-452](../../platinumRx/flexipill-core/src/pdp/pdp.service.ts#L448-L452)),
which the global filter returns without logging, and `SYSTEM_UNHEALTHY` is declared in the
TraceCodes enum but never emitted. So pods can be pulled out of service with no log line
anywhere. That alert has to come from k8s / Container Insights, not from CloudWatch Logs.

**Two design risks the alerts should make visible, but that are worth fixing:**

1. **Redis down disables every idempotency guard.** `RedisMutex.acquire()` returns `true`
   when Redis errors — deliberately, "to avoid blocking operations"
   ([mutex.ts:105-108](../../platinumRx/flexipill-core/src/utils/mutex.ts#L105-L108)).
   So during a Redis outage the app looks healthy while duplicate order placement,
   duplicate Sales Orders and double refunds all become possible. Treat a Redis alert as a
   data-integrity incident and reconcile afterwards, not as a caching blip.

2. **Redis down also fails readiness on every pod at once.** `checkHealth` gates on both
   `dbHealth` and `redisHealth`. Because Redis is shared, one blip pulls the entire fleet
   out of the Service — turning a degraded-but-working state into a total outage. Worth
   demoting Redis to an informational field in the readiness response.

Native CloudWatch metrics carry several of these with no code and none of the log-shape
caveats — RDS (`DatabaseConnections`, `CPUUtilization`, `FreeableMemory`), ElastiCache
(`CurrConnections`, `Evictions`), AmazonMQ (`ConsumerCount`, `MessageReadyCount`), SQS
(`ApproximateAgeOfOldestMessage`, DLQ depth). Prefer them over log-derived equivalents.

`infra-queue-consumer-crit` (`ConsumerCount = 0`) is strictly faster than the log-based
`so-backlog-crit` — minutes instead of up to an hour. Keep both; they fail differently.

## Latency

There is no true request-latency metric in the logs. `INITIATE_ORDER_STEP_TIMING` is
declared in the TraceCodes enum but **never emitted** anywhere in the codebase.

What exists:
- `REST_SLOW_CALL` — outbound HTTP over 3000ms (`RestServiceUtils.DEFAULT_SLOW_CALL_THRESHOLD_MS`, per-call overridable). Downstream latency, not ours.
- `DB_SLOW_QUERY` — **10% sampled** (`db-query-logger.service.ts:19`). Multiply counts by ~10 before reasoning about volume.

For real server-side p95 you want load-balancer metrics (`TargetResponseTime`,
`HTTPCode_Target_5XX_Count`, `RequestCount`). The k8s Services in this repo are plain
ClusterIP on :3000 — whatever terminates HTTP lives outside the repo. **Find out what
it is.** If there is an ALB, its CloudWatch metrics give you an accurate 5xx/4xx/latency
SLI for free, with no code change and none of the caveats above, and several rows in
this sheet become redundant.

## Order of work

Do **not** create all of these at once. For each row:

1. Create the metric filter only. Let it collect for 2 weeks.
2. Look at the actual distribution. Replace the proposed threshold with a real one.
3. Create the alarm. Move `status` to `live`.
4. Only after the critical rows have been quiet-but-correct for a month, add the warning rows.

Setting thresholds before baselining is the most common way this ends up muted.

## Progress

| Module | Flow | Rows | Status |
|---|---|---|---|
| platform | Unhandled 5xx | 2 | **blocked** — exception filter logs nothing |
| platform | Caught 5xx / 4xx rate, process crashes | 6 | proposed |
| infra | DB, DB capacity, Redis, readiness, pod restarts, queue consumers (6 flows) | 12 | proposed |
| zoho | SO creation (6 flows) | 12 | proposed — needs baselining |
| orders | Placement, volume drop, PG callback, status, edit, read, latency (7 flows) | 14 | proposed |
| paymentsv2 | — | — | not started |
| shipmentV2 | — | — | not started |
| procurement | — | — | not started |

**Code fixes this work surfaced, roughly in priority order:**

1. Restore the `logError` call in `ErrorLoggingFilter` — without it there is no 5xx signal at all.
2. **Add a global interceptor stamping `route` (template, not `req.url`), `statusCode` and duration.** Unlocks per-route 4xx/5xx for all 837 routes from one dimensioned metric filter, plus the request-latency metric that does not exist today. Highest leverage item here.
3. Stop mapping `QueryFailedError` to HTTP 400 with the raw PG message (wrong status, leaks schema).
4. Reconsider gating readiness on Redis — it converts a degraded state into a total outage.
5. Emit `SYSTEM_UNHEALTHY` from `checkHealth` so readiness failures are visible in logs.
6. Either emit `INITIATE_ORDER_STEP_TIMING` or delete it; it is dead enum weight today.
