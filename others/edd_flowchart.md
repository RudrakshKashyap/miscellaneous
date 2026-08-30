# EDD (Estimated Delivery Date) — Architecture and Calculation Flow

This document outlines the end-to-end flow of the EDD (Estimated Delivery Date) calculation process, specifically focusing on the **V3.1 (Production)** architecture.

The EDD module is responsible for computing the estimated delivery date for a given pincode and set of drugs. It determines the best warehouse to ship from, the optimal carrier, and the most efficient delivery lane (Standard, Next Day, or Same Day).

---

## 1. The Entry Point (Facade)

### `getEstimatedDeliveryDateWithFeatureFlag`
* **Goal**: Act as the public facade for the EDD service and route the request to the correct version.
* **What it does**: 
  * Guards against non-serviceable pincodes. If a pincode isn't mapped in `getPincodeEtaMapping`, it immediately returns a null-EDD response.
  * Checks if the `drugCodes` array is empty. If so, it early-exits.
  * Consults the `EddVersionGateService` to figure out which version of the logic to run (Production is currently **v3.1**).
  * Dispatches the request via `EddDispatcherService.execute()` to the appropriate version strategy.
  * Formats the final response for display (e.g., generating "Today, 8pm" display strings).

---

## 2. The Orchestrator

### `getEstimatedDeliveryDateV3`
* **Goal**: Orchestrate the entire v3 EDD calculation pipeline from start to finish.
* **What it does**: It acts as the "manager" function, calling the steps below in sequence: 1) Hydrate context, 2) Filter Warehouses, 3) Calculate Standard EDDs, 4) Select the final Delivery Lane, and 5) Build the final response.

---

## 3. Step-by-Step Pipeline

### Step 1: `hydrateContext`
* **Goal**: Gather all the necessary background data upfront.
* **What it does**: Loads data into the `EddCalculationContext` in parallel. This includes fetching drug metadata (to check for cold-chain items) and fetching SDD/NDD configurations for the requested pincode.

### Step 2: `executeWarehouseFilteringPipeline`
* **Goal**: Whittle down the list of all possible warehouses to a final list of eligible candidates.
* **What it does**: Passes the warehouses through three sequential filters (gates):
  1. `resolveInitialWarehouses`: Sorts warehouses by priority and enforces order-limit caps.
  2. `filterWarehousesByColdChain`: If the cart has cold-chain drugs, it strictly filters for warehouses that can meet the ClickPost Turnaround Time (TAT) requirements for cold-chain.
  3. `filterWarehousesByMdm`: Filters out warehouses where the requested drugs are hidden or unavailable.

### Step 3: `calculateWarehouseEdds` -> `computeWarehouseEdd`
* **Goal**: Calculate the baseline `STANDARD` lane Delivery Date for every eligible warehouse.
* **What it does**: For every warehouse that survived the filtering, it runs `computeWarehouseEdd`, which calls several important utility functions:

  * **`computeRfdTime`**: Computes the "Ready For Dispatch" (RFD) time. It adds buffer hours to the current time, applying a larger buffer if procurement is required.
  * **`getCarrierData`**: Calls the ClickPost API to get the recommended carrier and the estimated transit time (TAT) for this warehouse-to-pincode journey.
  * **`resolveCutoffTimesWithWaterfall`**: A smart fallback function to determine the courier's pickup cutoff time. If the courier doesn't have a `STANDARD` cutoff configured (or we missed it today), it "waterfalls" to check if they have a faster `NDD` or `SDD` cutoff we can use for tomorrow morning, rather than falling back to a late generic 4 PM default.
  * **`computeEddFromCutoffs`**: Converts the raw cutoff time string (e.g., "14:30") into a precise JavaScript `Date` object for the pickup. It then adds the ClickPost TAT hours to this pickup time, and applies any overnight buffers to calculate the final `STANDARD` EDD.

### Step 4: `selectDeliveryLane`
* **Goal**: Determine if a faster lane (`SDD` or `NDD`) can beat the baseline `STANDARD` EDD, and make the final selection of Warehouse + Courier + Lane.
* **What it does**: Calls the following sub-functions to make the decision:

  * **`checkSddNddEligibility` / `evaluateSddNddCandidates`**: Checks if the pincode is eligible for faster lanes. **In v3.1**, this function was upgraded: instead of only evaluating the single top-recommended (P0) courier, it evaluates **ALL** couriers capable of SDD. It calculates what the delivery date would be for every possible fast-lane combination.
  * **`compareSddNddCandidates` (inside `pickBestCandidate`)**: Takes all the valid fast-lane candidates and pits them head-to-head to find the absolute best one, using strict tie-breakers:
    * **Rule 1**: Earliest calendar day in IST wins.
    * **Rule 2**: If on the same day, prefer the one with same-day warehouse pickup.
    * **Rule 3**: If still tied, prefer the higher-priority warehouse.
    * **Rule 4**: If still tied, prefer the earliest exact timestamp.
  * **`selectFinalResult`**: The final showdown. It compares the absolute best Fast Lane option (SDD/NDD) against the absolute best `STANDARD` option. If the Fast Lane actually delivers on an earlier calendar day, it wins. If it doesn't, the system saves on premium shipping costs and defaults to `STANDARD`.

### Step 5: `buildEddResponse`
* **Goal**: Package the final chosen data to send back to the client.
* **What it does**: Assembles the chosen `minEta`, `maxEta`, the winning `warehouseId`, and cold-chain flags. If the caller requested it, it also attaches an `orderEddMeta` "journey" object, which traces exactly how this calculation was made (useful for debugging and order tracking).

---

## 4. Fallback Scenarios
The module handles edge cases gracefully, exiting early when necessary:
* **Null-EDD (Unserviceable)**: If the pincode isn't in the DB mapping.
* **Null-EDD (No Drugs)**: If the request is empty.
* **Recommendation Fallback**: If ClickPost fails to return any recommendations but the pincode is serviceable, it uses a generic system-wide TAT fallback.
* **Cold-Chain Non-Serviceable**: If the cart requires cold-chain shipping but no warehouse can fulfill the strict transit-time requirements.
