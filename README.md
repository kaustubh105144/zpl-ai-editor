---

# ZPL AI Editor
### Bringing Natural Language AI to Mission-Critical Label Operations

---

## Why This Exists

In manufacturing and supply chains, a label is not a sticker. It is a compliance document, a traceability record, and an operational instruction — all in one. Get it wrong and you delay shipments, fail audits, or pull product off the line.

Yet the process of changing one looks like a 2003 IT helpdesk workflow.

Production spots an issue. Raises a ticket. IT edits raw ZPL code. Shares a preview. Production prints physically to verify. Mismatch found. Repeat. **Average turnaround: 2–3 days — for a font size change.**

I built this PoC to answer one question: *can we give production teams the ability to make label changes themselves — without ever compromising the controls that make labels trustworthy?*

---

## The Answer

Yes. But only if you design governance in from day one — not as an afterthought.

This platform lets a production operator type "move the QR code down 5mm and update the expiry year" in plain English. Amazon Bedrock generates the updated ZPL code instantly. The operator sees the original and modified label side by side, saves it, and prints it physically to verify. IT then reviews and approves before anything goes near production.

**The operator never touches ZPL. IT never loses control. The AI never acts autonomously.**

---

## What I Designed

The hardest part of this project was not the AI integration. It was the constraint system around it.

Anyone can wire an LLM to a text box. The real design challenge was: how do you let non-technical users modify a compliance-critical technical artifact — at speed — without creating risk?

The answer required four deliberate decisions:

**Immutable baselines.** IT-approved templates are locked. The AI works on a copy, always. The original cannot be overwritten by anyone below admin level.

**Output validation before display.** Every ZPL response from the LLM passes a rule-based quality gate — structural completeness, format integrity, policy compliance — before the user ever sees it. Bad outputs are rejected and regenerated automatically.

**Separation of duties in the data model.** Production users cannot approve their own changes. This is not a UI restriction. It is enforced at the database level.

**Zero path to production.** The system has no connectivity to live production environments. Promotion is a deliberate, manual, IT-controlled action — every time.

---

## Impact

| | Before | After |
|---|---|---|
| Label change turnaround | 2–3 days | 20–30 minutes |
| IT involvement per change | Required | Approval only |
| IT workload on label tasks | High | ↓ 60–70% |
| Misprint & rejection rate | High | ↓ 50–70% |
| Process cost | Baseline | ↓ 30–40% |

---

## Screenshots

*(See `/screenshots` folder)*

`01 — Login` · `02 — Admin Dashboard` · `03 — AI Chat` · `04 — Side-by-Side Preview`

---

## The Bigger Point

This project is not really about ZPL.

It is about a pattern that applies across any enterprise domain where a technical artifact needs to be modified by non-technical users under compliance constraints — configuration files, regulatory templates, pricing rules, contract clauses.

The architecture of *constrained AI with human oversight* is the reusable idea. Label management was the problem worth proving it on first.

---
## Stack

<sup><sub>Amazon Bedrock · AWS EC2 · AWS ALB · Route53 · AWS WAF · RDS PostgreSQL · ElastiCache Redis · AWS Secrets Manager · S3 + CloudFront · Labelary API</sub></sup>
---

*Proof of Concept — isolated test environment, no connectivity to live production systems.*

---
