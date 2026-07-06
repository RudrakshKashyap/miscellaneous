TypeORM's .leftJoin() has two forms:

By relation: .leftJoin('orders.reviews', 'r') — requires a @OneToMany/@ManyToOne on the entity.
By table name + custom ON condition: .leftJoin('reviews', 'r', '...condition...') — pure SQL join, no relation needed.

The @Module({ imports: [X] }) decorator evaluates its imports array eagerly, at the moment the module file is parsed by Node. If X is undefined at that exact moment, the imports array literally contains undefined, and no amount of forwardRef elsewhere fixes it after the fact.

1. Node starts loading ordersModule.ts
2. Sees `import ReviewsModule` → starts loading reviewsModule.ts
3. ReviewsModule.ts sees `import OrdersModule` → but OrdersModule is mid-load
   so its export is currently `undefined`
4. ReviewsModule.ts continues. Its `@Module` decorator runs.
   imports array evaluates → contains `undefined` (because OrdersModule is undefined right now)
5. ReviewsModule finishes loading.
6. Back in OrdersModule.ts → ReviewsModule is now fully defined.
   @Module decorator runs. imports array contains the (now valid) ReviewsModule.

forwardRef(() => X) is different — it doesn't try to read X at decoration time. It stores the function in the imports array. Nest calls the function later (during DI scanning), by which time both files have finished loading.

So for a mutual cycle (A imports B, B imports A), both decorators evaluate eagerly and both need their cross-module reference wrapped in forwardRef. Otherwise whichever side gets evaluated while the other is mid-load captures undefined.

One nuance — if it really were a chain cycle (A → B → C → A) rather than a mutual cycle, you'd only need forwardRef on one edge (the back-edge that closes the loop). Earlier when ReviewsModule only consumed AuthModule (no back-reference from Auth), a single forwardRef(() => AuthModule) was enough — that broke a longer chain. But here, OrdersModule and ReviewsModule literally appear in each other's imports list, so it's a mutual cycle, hence two forwardRefs.

module-graph resolution and provider DI resolution. They're different mechanisms and need separate fixes.

Layer 1: Module-graph forwardRef (in imports)

Rule of thumb:

Mutual cycle (A's imports list contains B, B's imports list contains A): forwardRef on BOTH sides. One-sided isn't enough because whichever side loads first sees the other as undefined, and you don't know which loads first.
Chain cycle (A → B → C → A, where A doesn't directly import C): forwardRef on the back-edge only (the one that closes the loop). The forward edges in the chain aren't circular references, so they resolve fine without it.

Layer 2: Provider DI forwardRef (in constructors)
This is a completely separate mechanism from module imports. Even after modules load, Nest still needs to wire providers (services) together. Constructor parameter types come from TypeScript's design:paramtypes reflection metadata, which captures class references at decoration time.

Same problem as module imports: if OrdersService is undefined when ReviewsService's constructor metadata is generated (because of TS file load order), then paramtypes captures undefined, and Nest's DI resolver throws "Cannot resolve dependencies".

@Inject(forwardRef(() => OrdersService)) overrides paramtypes with a lazy lookup token. Nest unwraps it later.

---

Both have the same fundamental flaw: they hide the dependency, they don't eliminate it. The cycle isn't a mechanical problem (about how the call is made) — it's a semantic problem (Orders genuinely needs review data, Reviews genuinely needs order data). Changing the call mechanism doesn't change that fact.

Let me walk through both concretely.

```ts
// In OrdersService, when building order response:
const correlationId = randomUUID();
const review = await new Promise((resolve) => {
  this.eventEmitter.once(`reviews.fetched.${correlationId}`, resolve);
  this.eventEmitter.emit('reviews.fetch', { orderId, userId, correlationId });
});

// In ReviewsService:
@OnEvent('reviews.fetch')
async handleFetch({ orderId, userId, correlationId }) {
  const review = await this.reviewDao.findByUserAndEntity(userId, ..., orderId);
  this.eventEmitter.emit(`reviews.fetched.${correlationId}`, review);
}
```

✅ DI graph no longer shows OrdersService → ReviewsService. No forwardRef in injection.
❌ Both modules still need EventEmitterModule imported. The dependency moved, didn't disappear.
❌ The reviews listener still injects ReviewDao to do the same work. Identical logic, more indirection.
❌ Type safety dies — event payloads typically pass as any or via shared interfaces that nothing enforces at the call site. If ReviewsService renames userId to userid in the listener, TypeScript can't catch it; you find out in prod.
❌ Stack traces don't cross event boundaries. "Why is the order response slow?" becomes much harder to debug.
❌ Silent failure mode: if no one listens for 'reviews.fetch' (because the module wasn't loaded, or someone deleted the listener), Orders hangs on the Promise forever (or times out, if you add a timeout).
❌ Latency overhead — each emit goes through the EventEmitter's internal dispatch.

Why people reach for these (and why they shouldn't)
It feels like decoupling because you can no longer trace Orders → Reviews in the DI graph. But that's not real decoupling — that's just invisibility. The runtime call still happens, the dependency still exists, the modules still need each other for the application to function. You've removed the compiler's ability to verify the contract; you haven't removed the contract.

The litmus test:

Decoupling that helps: two systems become deployable/scalable independently (e.g., extracting a billing service that runs on a different cluster).
Decoupling that hurts: same process, same repo, same deploy, same types — but with a transport layer between two function calls that don't need it.

Middleware runs before the AuthGuard.
