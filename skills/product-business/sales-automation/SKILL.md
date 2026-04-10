---
name: sales-automation
description: Build automated sales systems. Covers outbound sequences, pipeline management, CRM automation, lead scoring, cold outreach, and follow-up workflows.
---

# Sales Automation

Build systems that generate, qualify, and convert leads without manual effort per prospect.

---

## 1. Cold Outreach Sequence (5-touch)

```
Day 1:  Email 1 — Personalized intro + specific value prop
Day 3:  Email 2 — Different angle / case study
Day 7:  Email 3 — Useful resource relevant to their role
Day 12: Email 4 — Short follow-up ("Circling back")
Day 20: Email 5 — Break-up email ("I won't bother you again, but...")
```

### Email formulas

**Email 1 — Permission + value:**
```
Subject: [Specific observation about their company]

Hi [First name],

I noticed [specific thing about their company — job posting, product launch, funding news].

[One sentence connecting their situation to your solution].

We helped [similar company] [specific result in their world].

Worth 15 minutes to see if we could do the same for you?

[Your name]
```

**Email 5 — Break-up:**
```
Subject: Closing the loop

Hi [Name],

I've reached out a few times without hearing back. I'll take the hint.

If the timing is ever right to [specific outcome], you know where to find me.

Take care,
[Your name]
```

**Rule:** Never send >5 touches per sequence. Re-engage cold leads only after 90+ days with new angle.

---

## 2. Lead Scoring Model

```
Score each lead out of 100:

FIRMOGRAPHIC (40 points)
- Company size: 11–100 (20pts), 101–500 (15pts), 501+ (10pts), <10 (5pts)
- Industry: matches ICP (15pts), adjacent (8pts), off-ICP (0pts)
- Location: target regions (5pts), other (2pts)

BEHAVIORAL (40 points)
- Visited pricing page: 20pts
- Opened 3+ emails: 15pts
- Clicked a link: 10pts
- Replied to outreach: 40pts (auto-qualify)

DEMOGRAPHIC (20 points)
- Title: VP/Director (20pts), Manager (10pts), IC (5pts), Unknown (0pts)

Action thresholds:
- 70+: immediate sales call
- 50–69: fast-track nurture sequence
- <50: slow nurture / low priority
```

---

## 3. CRM Automation

### HubSpot workflows

```
Trigger: Contact submits demo form
Actions:
  1. Assign owner (round-robin or by territory)
  2. Send confirmation email
  3. Create Deal in pipeline stage "Demo Requested"
  4. Notify owner via Slack
  5. Set reminder: follow up if no response in 48h
```

```
Trigger: Deal idle for 14 days in "Proposal Sent"
Actions:
  1. Send owner task: "Follow up with [contact]"
  2. Enroll contact in "stalled deal" email sequence
  3. Notify manager
```

### Salesforce flows
```
Process Builder: New Lead → Score Lead → Route by Score
- High score: create Task for SDR, set follow-up date = today
- Medium score: add to drip campaign
- Low score: add to nurture list, set follow-up date = 30 days
```

---

## 4. LinkedIn Outreach Automation

**Manual-automated hybrid (safest):**
```
Step 1: Connection request — personalized note (300 chars max)
  "Hi [Name], I noticed you [observation]. I work with [similar companies] on [outcome]. 
   Would love to connect."

Step 2 (after connect, Day 2): Value message
  Share relevant article, research, or insight (no pitch)

Step 3 (Day 5): Soft ask
  "Given your focus on [topic], thought you might find [resource] valuable. 
   Are you currently exploring [problem area]?"

Step 4 (Day 12): Direct ask
  "Happy to share how [similar company] [achieved result]. 
   Worth 15 minutes?"
```

**Tools:** Lemlist, Apollo, Instantly, Clay (enrichment + sequencing)

---

## 5. Pipeline Management

**Stage definitions (customize per business):**
```
1. Prospecting       — Identified, not yet contacted (0%)
2. Connected         — Responded to outreach (10%)
3. Qualified         — BANT confirmed (25%)
4. Demo Scheduled    — Meeting booked (40%)
5. Demo Complete     — Needs sent (60%)
6. Proposal Sent     — Proposal reviewed (75%)
7. Negotiation       — Contract review (90%)
8. Closed Won        — Signed (100%)
9. Closed Lost       — Not moving forward
```

**BANT qualification:**
- **Budget**: Do they have budget for this category?
- **Authority**: Are you talking to the decision maker?
- **Need**: Is this solving a real, urgent problem?
- **Timeline**: When do they plan to decide/buy?

---

## 6. Follow-Up Automation Rules

- **Speed to lead**: Contact inbound leads within 5 minutes (conversion drops 80% after 30 min)
- **Persistence**: 80% of sales happen on the 5th–12th contact
- **Multi-channel**: Email → LinkedIn → Phone (each channel adds +15% response rate)
- **Personalization at scale**: Use `{{company}}`, `{{pain_point}}`, `{{recent_news}}` tokens
- **Unsubscribe handling**: Immediate removal, no re-contact for 12 months

---

## 7. Metrics

| Metric | Benchmark | Action if below |
|---|---|---|
| Open rate (cold email) | >40% | Improve subject lines |
| Reply rate (cold email) | >5% | Improve personalization, offer |
| Meeting booked rate | >10% of replies | Improve CTA, social proof |
| Show rate | >80% | Confirmation sequence, reminders |
| Close rate | >20% | Qualify better, improve demo |
| Sales cycle | Benchmark vs industry | Identify and remove blockers |
