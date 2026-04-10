---
name: resume-tailoring
description: Tailor resumes for specific job descriptions. Analyzes JD keywords, highlights relevant experience, rewrites bullet points for impact, and optimizes for ATS systems.
---

# Resume Tailoring

Match a resume to a specific job description to maximize interview conversion.

---

## 1. Job Description Analysis

Parse the JD into these categories:

```
REQUIRED skills (must match): [list]
PREFERRED skills (nice to have): [list]
Key responsibilities: [top 3–5]
Seniority signals: [years experience, scope of ownership]
Culture keywords: [fast-paced, cross-functional, data-driven, etc.]
ATS keywords: [exact phrases used 2+ times in JD]
```

**Rule:** Every required skill must appear in the resume, using the JD's exact phrasing when possible (ATS keyword matching).

---

## 2. Resume Structure

```
Header: Name | Email | LinkedIn | GitHub/Portfolio | Location (city only)

Summary (3–4 lines): 
  [Role title] with [X] years in [domain]. 
  Specialized in [2–3 skills matching JD]. 
  Track record of [one outcome statement].

Skills section: Match JD keywords directly
  Languages: [list]
  Frameworks: [list]
  Tools/Platforms: [list]
  Methodologies: [list]

Experience (reverse chronological):
  [Company] | [Title] | [Start – End]
  • [Achievement bullet] (see §3)

Education:
  [Degree] | [School] | [Year]

Certifications (if relevant):
  [Name] | [Issuer] | [Year]
```

---

## 3. Achievement Bullet Formulas

**Rule:** Every bullet = Action Verb + Specific Task + Quantified Result

```
Formula: "[Strong verb] [what you did] [for whom/what] resulting in [metric]"

Examples:
✅ "Reduced API response time by 40% by implementing Redis caching for user session data"
✅ "Led migration of 3 legacy services to microservices architecture, cutting deploy time from 2h to 8 min"
✅ "Built React component library used across 5 product teams, eliminating 60% of UI code duplication"

❌ "Worked on backend services" (no metric, no impact)
❌ "Responsible for maintaining the codebase" (passive, no achievement)
❌ "Helped improve performance" (vague, no ownership)
```

**Strong verb bank by function:**
- Engineering: Built, Architected, Optimized, Migrated, Refactored, Automated, Shipped
- Leadership: Led, Mentored, Hired, Established, Grew, Aligned
- Analysis: Identified, Analyzed, Reduced, Improved, Benchmarked
- Product: Designed, Launched, Delivered, Prioritized, Defined

---

## 4. ATS Optimization

```
ATS systems scan for:
1. Exact keyword matches from the job description
2. Job title alignment (your last title should match or be close to the target)
3. Education requirements (listed clearly)
4. Years of experience (clear dates, no gaps unexplained)

ATS killers (avoid):
✗ Tables (ATS can't parse them)
✗ Headers/footers with contact info (ATS skips them)
✗ Images, icons, graphics
✗ Columns/multi-column layouts (PDF columns parse wrong)
✗ Abbreviations without spelling out (ATS may not know "K8s" = "Kubernetes")
✗ File format: always submit as .docx or PDF (not .pages)
```

**Keyword density check:**
Take the JD's 10 most important keywords. Count how many times each appears in your resume. Every required keyword should appear at least once.

---

## 5. Tailoring by Role Level

### Individual Contributor
- Focus: technical depth, specific technologies, output metrics
- Bullet emphasis: what you built, performance improvements, code quality

### Tech Lead / Senior
- Focus: technical decisions + team impact
- Add: mentoring X engineers, technical roadmap, cross-team coordination
- Metrics: team velocity, reliability improvements, onboarding time

### Engineering Manager / Director
- Focus: org impact, hiring, retention, business outcomes
- Add: team size grown, hiring N engineers, budget owned
- De-emphasize: individual coding tasks (unless relevant)

---

## 6. Checklist Before Submitting

- [ ] Every required JD keyword appears in resume
- [ ] Job title on resume matches or is close to target title
- [ ] All bullets have a metric or specific outcome
- [ ] No gaps in employment unexplained (add freelance, startup, sabbatical label)
- [ ] Skills section matches JD technology stack
- [ ] Contact info in main body (not header/footer)
- [ ] No tables, columns, or images
- [ ] File is PDF (unless docx specifically requested)
- [ ] File name: `FirstLast-Resume-CompanyName.pdf`
- [ ] Total length: 1 page (0–3 years), 2 pages (3–10 years), max 3 pages (10+ years)
