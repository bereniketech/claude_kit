---
name: customer-success
description: Build customer success systems. Covers onboarding workflows, health scoring, renewal automation, churn prevention, support escalation, and knowledge base design.
---

# Customer Success

Build systems that retain customers, drive expansion, and turn users into advocates.

---

## 1. Onboarding Workflow

### Time-to-value framework
Define: What is the first "aha moment" customers experience? (e.g., first integration connected, first report generated, first sale made)

**Target: Reach aha moment within 72 hours of signup.**

```
Day 0: Welcome email + single next action (not a checklist)
  "Your one task: [connect integration / invite team / run first X]"

Day 1: Check-in (automated) — "Did you complete [action]? Here's what's next."

Day 3: Human touchpoint (for high-value accounts) — brief call or personalized email

Day 7: "You've been with us a week" — share what they've accomplished, suggest next step

Day 14: Case study from similar company — reinforce value, seed expansion

Day 30: Success check-in — are they hitting their goals?
```

---

## 2. Health Score Model

```
Score each customer weekly (0–100):

USAGE (40pts)
- Active users / seats purchased: 80%+ (20pts), 50–79% (10pts), <50% (0pts)
- Feature adoption: using 3+ core features (20pts), 1–2 (10pts), 1 only (0pts)

ENGAGEMENT (30pts)
- Logged in last 7 days (10pts)
- Opened last 3 emails (10pts)
- Attended training/webinar (10pts)

BUSINESS (20pts)
- Contract renewal >90 days away (20pts), <90 days (10pts), <30 days (5pts)
- NPS score >8 (20pts), 7–8 (10pts), <7 (0pts)

SUPPORT (10pts)
- No open critical tickets (10pts)
- Open non-critical tickets only (5pts)
- Open critical ticket (0pts)

Health status:
- Green: 70–100
- Yellow: 40–69 (at-risk, needs attention)
- Red: 0–39 (churn risk, escalate immediately)
```

---

## 3. Churn Prevention Playbooks

### Yellow account playbook (40–69 health)
```
Day 1: CS automated email — "We noticed you haven't [specific action]. Here's a quick guide."
Day 3: CS manual check — review account, identify specific adoption gap
Day 5: Personalized email with 1 specific recommendation
Day 10: Offer: free training session or success call
Day 14: Executive touch for >$10K ARR accounts
```

### Red account playbook (<40 health)
```
Immediate: Alert account owner + manager
Day 1: Discovery call — understand root cause (usage? ROI? alternative?)
Day 3: "Save the account" plan — propose changes, new use case, pricing adjustment
Day 7: Executive call (for high-value accounts)
Day 14: Decision point — commit to save or accept churn gracefully
```

---

## 4. Renewal Automation

```
180 days before renewal:
- Send: Annual review report (usage stats, value delivered, ROI)
- Action: Schedule QBR (Quarterly Business Review)

90 days before renewal:
- Send: Renewal reminder + upsell recommendation
- Action: CS creates renewal opportunity in CRM

60 days before renewal:
- Send: Case study from similar company that expanded
- Action: Present new features/tier to justify upgrade

30 days before renewal:
- Send: Final reminder with clear renewal CTA
- Action: Escalate non-responders to manager

14 days before:
- Send: "Your subscription renews in 14 days" — confirm payment method
- Action: Legal sends contracts for manual renewals
```

---

## 5. Support Escalation Matrix

```
Tier 1 (self-service):
- Knowledge base article exists
- In-app help widget
- Response: immediate (bot)

Tier 2 (support team):
- Bug or configuration issue
- Response SLA: 4h (paying), 24h (free)
- Owner: Support rep

Tier 3 (CS escalation):
- Repeated issue, customer frustration
- Feature request blocking renewal
- Response SLA: 2h for any-tier
- Owner: Customer Success Manager

Tier 4 (executive escalation):
- Churn threat from high-value account
- Legal/security/compliance issue
- Response SLA: 1h
- Owner: VP CS or CEO
```

---

## 6. Knowledge Base Design

```
Structure:
├── Getting Started
│   ├── Quick start guide (5 min to first value)
│   ├── Setup checklist
│   └── Video walkthrough
├── Core Features (one article per feature)
│   ├── [Feature 1] — what it does, how to use, examples
│   └── [Feature 2] — ...
├── Integrations
│   └── [Integration name] — setup + troubleshooting per integration
├── Troubleshooting
│   └── Common errors + fixes
└── FAQs
    └── Billing, plans, permissions, data
```

**Article quality rules:**
- Title answers the user's question ("How to connect Zapier" not "Zapier integration")
- First paragraph: one-sentence answer to the title question
- Steps: numbered, each step is one action
- Screenshot per major step
- End with: "Still having trouble? Contact support at [link]"
- Review and update every 90 days

---

## 7. Customer Success Metrics

| Metric | Target | Action if below |
|---|---|---|
| Net Revenue Retention (NRR) | >110% | Improve expansion motions |
| Gross Revenue Retention (GRR) | >90% | Focus on churn prevention |
| Time to first value | <72h | Simplify onboarding |
| CSAT / NPS | >50 NPS | Interview detractors |
| Support ticket volume / customer | Decreasing | Improve self-service |
| Onboarding completion rate | >80% | Remove friction from steps |
