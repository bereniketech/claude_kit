---
name: logistics-supply-chain
description: Manage carrier relationships, demand forecasting, inventory replenishment, freight exceptions, production scheduling, and returns across the full supply chain lifecycle.
---

# Logistics & Supply Chain

End-to-end supply chain operations covering carrier management, inventory planning, exception handling, production scheduling, and reverse logistics.

---

## 1. Carrier Relationship Management

**Source, vet, and negotiate with freight carriers across truckload (TL), LTL, intermodal, and brokerage.**

Every freight rate has components to negotiate independently — bundling obscures overpayment:
- **Base linehaul rate:** Per-mile or flat rate. Benchmark against DAT or Greenscreens lane rates.
- **Fuel surcharge (FSC):** Negotiate the FSC table (base trigger, increment, index lag), not just the current rate.
- **Accessorials:** Detention ($50–$100/hr after 2 free hours standard), liftgate ($75–$150), residential ($75–$125).
- **Contract vs. spot:** Target 75–85% contract freight. More than 30% spot means your routing guide is failing.

**Carrier scorecard — track 5 metrics, not 20:**

| Metric | Target | Red Flag |
|--------|--------|----------|
| On-time delivery (OTD) | ≥95% | <90% |
| Tender acceptance rate | ≥90% (primary) | <80% |
| Claims ratio (% of spend) | <0.5% | >1.0% |
| Invoice accuracy | ≥97% | <93% |
| Tender-to-pickup time | Within 2 hrs (FTL) | Consistent late pickup |

**FMCSA compliance:** Verify active MC authority, insurance minimums ($1M — above FMCSA's $750K minimum), safety rating (never Unsatisfactory), and broker bond ($75K) before first load and quarterly thereafter.

**Portfolio strategy:** 60–70% asset carriers, 20–30% brokers, 5–15% specialty. Build 3-deep routing guides for lanes with >2 loads/week. No single carrier exceeds 40% of any critical lane.

**RFP process (8–12 weeks):** Analyze 12 months of lane data → lane-level bid packages (not portfolio bids) → evaluate on 40–50% cost / 25–30% service history / 15–20% capacity commitment → award in waves → 30-day parallel period.

**Carrier exit triggers** (after documented corrective action): OTD <85% for 60 days, tender acceptance <70% for 30 days, claims ratio >2% for 90 days, FMCSA lapse, double-brokering confirmed.

---

## 2. Inventory and Demand Planning

**Translate commercial intent into executable purchase orders while minimizing stockouts and excess inventory.**

**Forecasting method selection:**

| Demand Pattern | Primary Method | Review Trigger |
|---------------|----------------|----------------|
| Stable, high-volume | Weighted moving average (4–8 weeks) | WMAPE >25% for 4 weeks |
| Trending | Holt's double exponential smoothing | Tracking signal exceeds ±4 |
| Seasonal | Holt-Winters (multiplicative) | Season correlation <0.7 |
| Intermittent (>30% zero periods) | Croston's method | Mean inter-demand interval shifts >30% |
| Promotion-driven | Causal regression (baseline + lift layer) | Post-promo actuals deviate >40% |

**Safety stock:** `SS = Z × σ_d × √(LT + RP)`. For lead time variability: `SS = Z × √(LT_avg × σ_d² + d_avg² × σ_LT²)`. Service level targets: A-items 95–97.5%, B-items 95%, C-items 90–92%, CZ 85%.

**ABC/XYZ classification:** A = top 20% SKUs driving 80% revenue. X = CV <0.5 (predictable). AX: automated replenishment with tight SS. AZ: human review every cycle. CZ: candidate for discontinuation.

**Promotional lift:** Strip promo volume from baseline before fitting models. Typical lifts: 15–40% for price reduction only, 80–200% for TPR + display + circular, 300–500%+ for doorbusters. Always model post-promo dip (default: 40% of incremental lift, concentrated in week 1).

**Forecast accuracy metrics:** WMAPE <25% target, >35% red flag. Bias ±5% healthy, >±10% for 4 weeks signals structural model problem. Tracking signal exceeds ±4 → re-parameterize or switch methods.

**Escalation triggers:** Projected stockout on A-item within 7 days → alert within 4 hours. Vendor lead time increase >25% → notify within 1 business day. Promo forecast miss >40% → debrief within 1 week.

---

## 3. Logistics Exception Management

**Resolve freight exceptions — delays, damages, losses, and carrier disputes — quickly while protecting financial interests.**

**Exception taxonomy:** Delay (transit) ~40% of all exceptions. Damage (visible, concealed, temperature). Shortage / overage. Refused delivery. Lost (full or partial shipment). Contaminated.

**Claims process:**
- **Carmack Amendment (US domestic):** Carrier liable for actual loss. Shipper must prove: goods in good condition at tender, arrived damaged/short, amount of damages.
- **Filing deadline:** 9 months from delivery date (49 USC § 14706). Miss it and the claim is time-barred.
- **Required docs:** Original BOL, delivery receipt with exception noted, commercial invoice, inspection report, photographs, repair estimates.
- **Carrier response:** 30 days to acknowledge, 120 days to pay or decline.

**Eat-the-cost vs. fight-the-claim thresholds:**
- <$500 and strong carrier relationship → absorb (admin cost exceeds recovery)
- $500–$2,500 → file, accept settlements >70%
- $2,500–$10,000 → full claims process, reject <80%
- >$10,000 → VP-level awareness, reject <90%, legal review if denied
- Any amount + 3rd+ exception from same carrier in 30 days → treat as carrier performance issue

**Priority sequencing** (multiple active exceptions): (1) Safety/regulatory, (2) Production shutdown risk, (3) Perishable <48 hours shelf life, (4) Highest financial impact by customer tier, (5) Oldest unresolved exception.

**Exception rate targets:** <25 per 1,000 shipments (red flag: >40). Mean resolution time <72 hours (red flag: >120). Financial recovery rate >75% (red flag: <50%).

---

## 4. Production Scheduling

**Sequence jobs, balance lines, optimize changeovers, and resolve disruptions to maximize constraint throughput.**

**Theory of Constraints (DBR):** Identify the drum (constraint — work centre with highest load/capacity ratio, typically >85%). Buffer time = 50% of production lead time for the constraint operation. The rope limits new work release to the constraint's rate. A minute lost at the constraint is a minute lost for the entire plant.

**Buffer zones:** Green (<33% consumed) = well-protected. Yellow (33–67%) = expedite upstream. Red (>67%) = immediate management action, potential overtime upstream.

**Job priority sequencing:**
1. Past-due jobs first (ordered by penalty exposure)
2. Jobs feeding the constraint when buffer is yellow/red
3. Dispatching rule by product mix: EDD for high-variety short-run, SPT for long-run few products, setup-aware EDD for mixed with sequence-dependent setups
4. Tie-breaker: higher customer tier, then higher margin

**Changeover optimization:** Build setup matrix (changeover time and cost for each product pair). Apply nearest-neighbour heuristic then 2-opt swaps. Validate against due dates — due date compliance trumps changeover optimization.

**SMED methodology:** Classify setup elements as internal (machine stopped) or external (machine running). Convert internal to external where possible. Typical result: 40–60% setup time reduction from reclassification alone.

**Disruption response** (within 30 minutes): Assess impact window → freeze jobs in-process or within 2 hours of start → re-sequence unlocked jobs using priority framework → communicate revised schedule → lock for minimum 4 hours.

**OEE:** Availability × Performance × Quality. World-class: 85%+. Typical discrete manufacturing: 55–65%. A 2% yield improvement at the constraint equals a 2% capacity expansion.

**Key performance targets:**

| Metric | Target | Red Flag |
|--------|--------|----------|
| Schedule adherence (±1 hr) | >90% | <80% |
| On-time delivery | >95% | <90% |
| OEE at constraint | >75% | <65% |
| Constraint utilization | >85% | <75% |
| Unplanned downtime | <5% | >10% |

---

## 5. Returns and Reverse Logistics

**Authorize, inspect, disposition, and recover value from returned goods while detecting fraud and managing vendor recovery.**

**Condition grading:**
- **Grade A (Like New):** Original packaging intact, all accessories, passes functional test → restock as new (85–100% recovery)
- **Grade B (Good):** Minor cosmetic wear, functional → open box / renewed (60–80% recovery)
- **Grade C (Fair):** Visible wear, missing minor accessories → liquidate or refurbish if cost <20% of recovered value (30–50%)
- **Grade D (Salvage):** Non-functional or heavily damaged → parts harvest or recycle (5–15%)

**Disposition routing by category:**

| Category | Grade A | Grade B | Grade C | Grade D |
|----------|---------|---------|---------|---------|
| Consumer Electronics | Restock (test first) | Open box / Renewed | Refurb if ROI >40%, else liquidate | Parts or e-waste |
| Apparel | Restock if tags on | Repackage / outlet | Liquidate by weight | Textile recycling |
| Health & Beauty | Restock if sealed | Destroy | Destroy | Destroy |

**Fraud scoring** — flag for review at 65+, hold refund at 80+:
- Serial number mismatch on electronics: +40 points
- Product weight differs >5% from expected: +25 points
- Return rate >30% (rolling 12 months): +15 points
- No-receipt return: +15 points
- New account (<30 days old): +10 points

**Vendor recovery:** Pursue claims >$500. Batch claims $200–$500. Offset claims <$200 against next PO. Raise threshold to $1,000 for overseas vendors. RTV claim window is typically 90 days from receipt — do not let eligible product sit past this window.

**Key metrics:**

| Metric | Target | Red Flag |
|--------|--------|----------|
| Processing time (receipt to refund) | <48 hours | >96 hours |
| Restock rate | >45% | <30% |
| Fraud detection rate | >80% | <60% |
| Vendor recovery rate | >70% | <45% |
| Cost per return processed | <$8.00 | >$15.00 |

**Automatic escalation:** Return value >$5,000 → supervisor approval before refund. Fraud score ≥80 → hold and route to fraud review immediately. Recalled product identified → do not process as standard return.
