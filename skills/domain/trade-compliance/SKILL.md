---
name: trade-compliance
description: Classify goods under HS/HTS codes, optimize duties, screen restricted parties, prepare customs documentation, manage non-conformances, and execute CAPA programs across regulated environments.
---

# Trade Compliance and Quality Non-Conformance

Lawful, cost-optimized cross-border trade combined with quality control — covering customs classification, duty optimization, restricted party screening, NCR lifecycle management, and CAPA execution.

---

## 1. HS Tariff Classification

Apply the General Rules of Interpretation (GRI) in strict order — never skip to a later rule when an earlier one resolves the classification.

**GRI sequence:**
- **GRI 1:** Heading text and Section/Chapter notes (~90% of classifications resolved here)
- **GRI 2(a):** Incomplete articles classified as complete if they have the essential character
- **GRI 2(b):** Mixtures classified by the material giving essential character
- **GRI 3(a):** Two candidate headings → prefer the more specific
- **GRI 3(b):** Composite goods or sets → classify by component giving essential character
- **GRI 3(c):** When 3(a) and 3(b) fail → heading that occurs last in numerical order
- **GRI 4:** Analogous goods (last resort)
- **GRI 6:** Subheading classification follows same principles within the relevant heading

**Classification workflow:**
1. Get full technical specification — material, function, dimensions, intended use. Never classify from a product name alone.
2. Determine Section and Chapter using notes (Chapter notes override heading text).
3. Apply GRI 1. If resolved, proceed to subheading level (GRI 6).
4. If multiple candidate headings, apply GRI 2 then GRI 3 in sequence.
5. Check binding rulings: CBP CROSS database, EU BTI database, WCO opinions.
6. Document the GRI applied, headings considered and rejected, and the determining factor.

**Common pitfalls:** Multi-function devices (classify by primary function per GRI 3(b)). Parts vs. accessories (Section XVI Note 2). Textile composites (weight percentage of fibres determines classification). Software on physical media (the medium classifies, not the software).

---

## 2. Customs Documentation

| Document | Key Requirements |
|----------|-----------------|
| Commercial Invoice | Seller/buyer, goods description, quantity, unit price, total value, currency, Incoterms, country of origin — must conform to 19 CFR §141.86 (US) |
| Packing List | Weight, dimensions, marks/numbers matching BOL, piece count |
| Certificate of Origin | USMCA: 9 required data elements per Article 5.2. EUR.1 for EU preferential. Form A for GSP. |
| ISF 10+2 (US) | Submit 24 hours before vessel loading. Late or inaccurate ISF: $5,000 per violation. |
| Entry Summary (CBP 7501) | File within 10 business days of entry. Legal declaration — errors create penalty exposure under 19 USC §1592. |

**Incoterms 2020 compliance implications:**
- **EXW:** Buyer becomes exporter of record in seller's country — rarely appropriate for international trade.
- **DDP:** Seller must register as importer of record or use non-resident importer arrangement.
- **CIP:** Now requires Institute Cargo Clauses (A) — all-risks coverage (change from 2010).
- **Valuation impact:** US CBP generally excludes international freight/insurance from transaction value; EU includes transport and insurance to place of entry. Never assume Incoterms determine customs value.

---

## 3. Duty Optimization

**FTA utilization:** Identify applicable FTAs → determine product-specific rule of origin from the FTA annex → trace all non-originating materials through the BOM → calculate RVC if required → apply cumulation rules → prepare and retain certification.

USMCA RVC methods: Transaction Value: `RVC = ((TV − VNM) / TV) × 100`. Net Cost: `RVC = ((NC − VNM) / NC) × 100`. Net cost method often yields higher RVC when margins are thin (excludes sales promotion, royalties, shipping from denominator).

**Other optimization tools:**
- **Foreign Trade Zones (FTZs):** Duty deferral, inverted tariff relief, no duty on waste/scrap or re-exports.
- **Duty Drawback:** Refund of 99% of duties paid on imports subsequently exported. File within 5 years of import.
- **Temporary Import Bonds (TIBs) / ATA Carnet:** Duty-free entry for professional equipment, samples, exhibition goods. Export within the statutory window or full duty plus penalties apply.

---

## 4. Restricted Party Screening

Screen all transaction parties: buyer, seller, consignee, end user, freight forwarder, banks, and intermediate consignees.

**Mandatory US lists:** SDN (OFAC), Entity List (BIS), Denied Persons List (BIS), Unverified List (BIS), Military End User List (BIS).

**Screening hit protocol:**
1. Assess match quality: name similarity, address correlation, country nexus, alias analysis, date of birth. Below 85% name similarity with no address or country correlation → likely false positive, document and clear.
2. Cross-reference against company registrations, D&B, website, and prior transaction history.
3. Check list specifics: SDN → OFAC licence required. Entity List → BIS licence with presumption of denial. Denied Persons List → absolute prohibition, no licence available.
4. Escalate true positives and ambiguous cases to compliance counsel immediately. Never proceed while a hit is unresolved.
5. Document: screening tool, date, match details, adjudication rationale, disposition. Retain 5 years minimum.

**Rule:** ~95% of screening hits are false positives. Adjudicate every hit — document the rationale.

---

## 5. Penalties and Prior Disclosure

**US penalty framework (19 USC §1592):**
- Negligence: 2× unpaid duties or 20% of dutiable value (mitigated to 1× or 10%)
- Gross negligence: 4× unpaid duties or 40% of dutiable value
- Fraud: full domestic value, potential criminal referral

**Prior disclosure (19 CFR §162.74):** Filing before CBP initiates investigation caps penalties at interest on unpaid duties (negligence) or 1× duties (gross negligence). Requirements: identify the violation, provide correct information, tender unpaid duties. Must file before CBP issues a pre-penalty notice.

**Record-keeping:** US 19 USC §1508 requires 5-year retention of all entry records.

---

## 6. Quality Non-Conformance (NCR) Lifecycle

Detect → Contain → Classify → Investigate → Disposition → CAPA → Verify effectiveness.

**Containment is always first.** Quarantine nonconforming material before root cause analysis begins. Physical segregation with red-tag in designated MRB area. Electronic hold in ERP to prevent inadvertent shipment.

**Disposition decision logic (apply in order):**
1. Safety/regulatory critical → do not use-as-is. Rework to full conformance or scrap.
2. Customer-specific requirements violated → contact customer for concession before disposing.
3. No functional impact + within material review authority → use-as-is with documented engineering justification.
4. Reworkable → rework if cost <60% of replacement cost, otherwise scrap.
5. Supplier-caused → RTV with SCAR.

**NCR disposition options:** Use-as-is (engineering justification required; aerospace needs customer approval), Rework (approved procedure, re-inspect to original spec), Repair (permanent deviation accepted, customer concession), Return to Vendor (SCAR, debit memo), Scrap (authorized sign-off, witness destruction for serialized/safety-critical parts).

---

## 7. Root Cause Analysis

**Rule:** Your root cause is not a root cause if it contains the word "error," if your corrective action is "retrain the operator," or if the root cause restates the problem.

**Method selection:**
- Single-event, simple causal chain → **5 Whys** (1–2 hours). Each "why" must be verified with data.
- Multiple potential cause categories → **Ishikawa (6M: Man, Machine, Material, Method, Measurement, Environment)** + 5 Whys on likely branches (4–8 hours).
- Recurring or process-related → **8D** with full team (20–40 hours, D0–D8). Required by most automotive OEMs.
- Safety-critical or high-severity → **Fault Tree Analysis** with quantitative risk assessment (40–80 hours). Required for aerospace and medical device events.

---

## 8. CAPA System

**Initiation triggers:** Repeat non-conformances (same failure mode 3+ times), customer complaints, audit findings, field failures, SPC signals, near-miss events. Over-initiating CAPAs dilutes resources; under-initiating creates audit findings.

**Effective CAPA writing:** Specific, measurable, addresses the verified root cause. Bad: "Improve inspection procedures." Good: "Add torque verification at Station 12 with calibrated torque wrench (±2%), documented on traveler checklist WI-4401 Rev C, effective 2025-04-15." Every CAPA needs owner, target date, and evidence of completion.

**Closure criteria:** Verify implementation (action completed as planned) AND validate effectiveness (defect rate dropped to zero over 90 days of production data, 3 production lots, or one audit cycle — whichever is most meaningful). Do not close a CAPA at verification only.

**Regulatory expectations:** FDA 21 CFR 820.100. IATF 16949 §10.2.3–10.2.6. AS9100 §10.2. ISO 13485 §8.5.2–8.5.3.

---

## 9. Statistical Process Control (SPC)

**Western Electric Rules** — investigate immediately on any signal:
- Rule 1: One point beyond 3σ
- Rule 2: Nine consecutive points on one side of center line
- Rule 3: Six consecutive points steadily increasing or decreasing
- Rule 4: Fourteen consecutive points alternating up and down

**Rule:** Do not react to common cause variation. Adjusting a stable process increases variation (tampering). Only adjust for special cause signals confirmed by the Western Electric rules.

**Capability targets:** IATF 16949 requires Cpk ≥1.33 for established processes, Ppk ≥1.67 for new processes. A process with Cp=2.0 but Cpk=0.8 is capable but not centered — fix the mean, not the variation.

---

## 10. Key Performance Targets

**Trade compliance:**

| Metric | Target | Red Flag |
|--------|--------|----------|
| Classification accuracy | >98% | <95% |
| FTA utilization (eligible shipments) | >90% | <70% |
| Entry rejection rate | <2% | >5% |
| CBP examination rate | <3% | >7% |
| Screening false positive adjudication time | <4 hours | >24 hours |

**Quality:**

| Metric | Target | Red Flag |
|--------|--------|----------|
| NCR closure time (median) | <15 business days | >30 business days |
| CAPA on-time closure | >90% | <75% |
| CAPA effectiveness (no recurrence) | >85% | <70% |
| Supplier PPM (incoming) | <500 PPM | >2,000 PPM |
| Customer complaint rate (per 1M units) | <50 | >200 |
