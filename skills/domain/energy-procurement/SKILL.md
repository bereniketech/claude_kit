---
name: energy-procurement
description: Procure electricity and gas, optimize tariffs and demand charges, evaluate renewable PPAs, hedge market risk, and report Scope 2 emissions across multi-facility portfolios.
---

# Energy Procurement

Full lifecycle energy management for large commercial and industrial (C&I) consumers — tariff optimization, competitive procurement, demand charge management, renewable sourcing, and sustainability reporting.

---

## 1. Electricity Bill Anatomy

Understand each cost component independently — bundling into a single "rate" hides optimization opportunities:

- **Energy charges (40–55% of bill):** Per-kWh cost. Flat rate, TOU, or real-time pricing. In deregulated markets, this is the competitively procured component.
- **Demand charges (20–40% of bill):** Billed on highest single 15-minute average kW in the billing period. One bad 15-minute interval can add $5,000–$15,000 to a monthly bill.
- **Capacity charges:** In markets with capacity obligations (PJM, ISO-NE, NYISO), your share is set by your Peak Load Contribution (PLC) during prior-year system peak hours. Reducing load during those 1–5 critical summer hours can cut capacity charges 15–30% the following year.
- **T&D charges:** Regulated, generally non-bypassable. Even with on-site generation, you pay distribution charges for being connected to the grid.
- **Riders and surcharges:** Track open PUC rate case proceedings — a filing can add $0.005–$0.015/kWh to delivered cost.

---

## 2. Load Profiling

Profile each facility using 15-minute interval meter data before any procurement or optimization decision:

- **Load factor** = (Total kWh) / (Peak kW × Hours). High load factor (>0.75): flat, predictable — buy around-the-clock blocks. Low load factor (<0.50): spiky peaks — demand charges dominate, peak shaving has highest ROI.
- **Base vs. variable load:** Base load runs 24/7 (refrigeration, servers, continuous manufacturing). Variable load correlates with production schedules, occupancy, and weather.
- **Typical manufacturing breakdown:** HVAC 25–35%, production motors 30–45%, compressed air 10–15%, lighting 5–10%. Compressed air systems often have the worst peak-to-average ratio.

---

## 3. Procurement Strategy

**Choose based on the company's tolerance for budget variance and the current market cycle:**

| Strategy | Description | Best For |
|----------|-------------|----------|
| Fixed-price | Locked $/kWh for 12–36 months. Pay 5–12% premium above forward curve. | Budget certainty over cost minimization |
| Index/variable | Pay real-time or day-ahead wholesale + small adder. Lowest average cost, full price spike exposure. | Companies with active risk management and tolerance for variance |
| Block-and-index | Fixed-price blocks for baseload (60–80%), remaining at index. | Balancing cost optimization with partial certainty |
| Layered procurement | Buy in tranches over 12–24 months (e.g., 25% per quarter). Dollar-cost averaging for energy. | Eliminating market timing risk — the single most effective risk management technique |

**Rule:** Never try to call the bottom on energy markets. Layer instead. When forwards are in the bottom quartile of the 5-year range, accelerate procurement. When in the top quartile, decelerate and increase index exposure.

**RFP process:** Issue to 5–8 qualified retail energy providers. Include 36 months of interval data, load factor, utility account numbers, contract expiration dates, and sustainability requirements. Evaluate on total cost, supplier credit quality (check S&P/Moody's — a bankruptcy forces you into utility default service), contract flexibility, and value-added services.

---

## 4. Demand Charge Management

**Identify top 10 demand-setting intervals per month from 15-minute interval data.** In most facilities, 6–8 of the top 10 peaks share a common root cause — simultaneous startup of large loads during morning ramp-up (6:00–9:00 AM).

**Mitigation options:**

- **Load shifting:** Move discretionary loads (batch processes, charging, thermal storage) to off-peak. A 500 kW shift saves $5,000–$12,500/month in demand charges alone.
- **Battery peak shaving:** 500 kW / 2 MWh battery costs $800K–$1.2M installed. At $15/kW demand charge, shaving 500 kW saves $90K/year. Stack with TOU arbitrage, capacity tag reduction, and demand response (DR) to reduce payback from 9–13 years to 5–7 years.
- **Demand response (DR) programs:** PJM Economic DR pays LMP for curtailed load. ERCOT ERS pays a standby fee plus energy payment. A 1 MW curtailment capability earns $15K–$80K/year.
- **Ratchet clause check:** Many tariffs bill demand at the higher of actual peak or 60–80% of the prior 11-month maximum. A single accidental peak locks elevated billing demand for a year. Always check for ratchet provisions before facility modifications.

---

## 5. Renewable Energy Procurement

**Physical PPA:** Contract directly with a renewable generator at a fixed $/MWh for 10–25 years. Receive energy and associated RECs. Manage: basis risk (generator node vs. load zone), curtailment risk, and shape risk (solar produces when the sun shines, not when you consume).

**Virtual PPA (VPPA):** A contract-for-differences. If market price > strike, generator pays you the difference. If market price < strike, you pay the generator. You receive RECs. Does not change your physical power supply. Requires ISDA agreement and mark-to-market accounting treatment — get CFO/treasury approval.

**RECs:** 1 REC = 1 MWh of renewable attributes. Unbundled RECs: $1–$5/MWh for wind, $5–$15/MWh for solar. Satisfies market-based GHG Protocol Scope 2 accounting but faces increasing scrutiny on additionality.

**On-site generation:** On-site solar PPA pricing $0.04–$0.08/kWh. Reduces T&D exposure and can lower capacity tags. Evaluate against net metering risk (compensation rate changes) and interconnection costs.

**PPA evaluation checklist:**
1. Does the project pencil against the forward curve for the full contract tenor? Model high/low gas price scenarios.
2. What is the basis risk? Require 5+ years of historical node-to-load-zone spread data.
3. What is the curtailment exposure? ERCOT curtails wind 3–8% annually, CAISO curtails solar 5–12% in spring. Negotiate curtailment cap or generator-bears-curtailment settlement structure.
4. What are the credit requirements? A $50M notional VPPA may require a $5–$10M letter of credit — factor LC cost into economics.

---

## 6. Market Structures

- **Regulated markets (~35% of US commercial load):** Single utility provides generation, T&D. Rates set by state PUC. Optimization limited to tariff selection, demand management, and on-site generation.
- **Deregulated markets:** Competitive generation. ISOs/RTOs: PJM (Mid-Atlantic/Midwest), ERCOT (Texas), CAISO (California), NYISO, ISO-NE, MISO, SPP.
- **LMP (Locational Marginal Pricing):** Wholesale prices vary by node reflecting generation costs, transmission losses, and congestion. LMP = Energy + Congestion + Loss components. Congestion can add $5–$30/MWh to delivered cost in constrained zones.

---

## 7. Risk Management

**Hedging:** Layered procurement is the primary hedge. Supplement with financial hedges for specific exposures. Put options on wholesale electricity (cost $2–$5/MWh premium) cap index pricing exposure against $200+/MWh spikes.

**Target hedge ratio:** 60–80% hedged / 20–40% index. Manufacturers with energy as a material input cost lean toward 80% hedged. Office portfolios with energy as overhead can tolerate more index exposure.

**Regulatory risks to monitor:** Tariff changes through rate cases, capacity market reform, carbon pricing legislation, net metering policy changes. Track open PUC proceedings for each utility in your portfolio.

---

## 8. Sustainability Reporting

**Scope 2 — two methods (GHG Protocol requires dual reporting):**
- **Location-based:** Average grid emission factor for your region (eGRID in US).
- **Market-based:** Reflects procurement choices — PPAs, VPPAs, RECs, utility green tariffs reduce market-based Scope 2.

**RE100:** Commit to 100% renewable electricity. Acceptable instruments: physical PPAs, VPPAs with RECs, utility green tariff programs, unbundled RECs (additionality requirements tightening), on-site generation.

**CDP/SBTi:** Energy procurement data feeds CDP Climate Change questionnaire Section C8. Locking in fossil-heavy supply for 10+ years can conflict with SBTi Science Based Targets trajectories.

---

## 9. Escalation Triggers and KPIs

| Trigger | Action | Timeline |
|---------|--------|----------|
| Wholesale prices exceed 2× budget for 5+ days | Notify finance, evaluate hedge position | Within 24 hours |
| Supplier credit downgrade below investment grade | Review termination provisions, assess replacement | Within 48 hours |
| Demand peak exceeds ratchet threshold >15% | Investigate with operations, model billing impact | Within 24 hours |
| Utility rate case filed with >10% proposed increase | Engage regulatory counsel | Within 1 week |
| Capacity tag increases >20% from prior year | Analyze coincident peak intervals, develop peak response plan | Within 2 weeks |
| Grid emergency / rolling blackouts | Activate emergency load curtailment | Immediate |

| KPI | Target | Red Flag |
|-----|--------|----------|
| Weighted avg energy cost vs. budget | Within ±5% | >10% variance |
| Procurement cost vs. forward curve at execution | Within 3% | >8% premium |
| Demand charges as % of total bill | <25% (manufacturing) | >35% |
| Renewable energy % (market-based Scope 2) | On track to RE100 target | >15% behind trajectory |
| Supplier contract renewal lead time | ≥90 days before expiry | <30 days before expiry |
| Budget forecast accuracy (Q1 vs. actuals) | Within ±7% | >12% miss |
