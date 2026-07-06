- Postgres always creates a backing index for a `PRIMARY KEY` and a `UNIQUE` constraint -- you don't need a separate `CREATE INDEX` for them.

- pg (node-postgres) returns Postgres `numeric`/`decimal` columns as strings to avoid JS float precision loss.
- `getRawMany()` bypasses any entity-level transformer, so even if the `Reviews` entity uses a string→float transformer (like Utils.stringToFloat), raw queries skip it.
- r.rating::float as rating casts the numeric column to float8 in Postgres, which pg returns as a JS number directly.

Why this happens: Postgres numeric/decimal types are arbitrary-precision, but JS number is 64-bit float — so pg deserializes them as strings to avoid silent precision loss.

```js
static stringToFloat = {
    // TypeORM Transformer
    to: (value: number | null) => value, // intercepts data right before it goes into the db
    // pg returns decimal column as string by default to avoid precision loss
    from: (value: string | null) => (value !== null ? parseFloat(value) : null), // and right after it comes out
  };
```

Fix -> Cast in SQL

```sql
'r.rating::float as rating'
```

The groupBy is needed because the SELECT uses json_agg(...) as warehouse_orders — an aggregate function. In Postgres, when you mix aggregate and non-aggregate columns in SELECT, every non-aggregate column must appear in GROUP BY.
Look at the query in orders.service.ts:4519-4573:

It LEFT JOINs warehouse_orders (wo) — a one-to-many relation (one parent order → many warehouse orders).
To collapse those multiple wo rows back into a single row per order, it wraps them with json_agg(json_build_object(...)) at line 4549-4564.
Once json_agg is in the SELECT, Postgres requires every other selected column (all the orders._, od._, sm.customer_status, r.rating) to be in GROUP BY — otherwise it errors with "column must appear in the GROUP BY clause or be used in an aggregate function."
That's why the GROUP BY repeats the entire column list from the SELECT.

Could it be simpler?
Yes — a couple of cleaner alternatives:

GROUP BY orders.order_id only (if you add a PRIMARY KEY shortcut). Postgres allows functional dependency: if you GROUP BY a primary key, all other columns from that table are implicitly grouped. But it only works per-table, so you'd still need od.\*, sm.customer_status, r.rating listed — unless order_details, status_master, reviews rows are unique per order_id (then add their PKs).

Move json_agg into a subquery / lateral join, e.g.:

LEFT JOIN LATERAL (
SELECT json_agg(json_build_object(...)) as warehouse_orders
FROM warehouse_orders wo
LEFT JOIN status_master wsm ON wo.status_id = wsm.status_id
WHERE wo.parent_order_id = orders.order_id
) wo_agg ON true
This eliminates the GROUP BY entirely — cleaner and usually faster.
