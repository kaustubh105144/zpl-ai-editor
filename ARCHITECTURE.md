---

# Architecture & Technical Design

---

## Design Principles

Before the flows — the decisions that shaped them.

- **DLP before AI.** User prompts are scanned for sensitive content before touching any AI model. Malicious input never reaches Bedrock.
- **Session-loaded templates.** Assigned ZPL templates are fetched from PostgreSQL at login and stored in Redis for the session. Every AI request reads from Redis — not the database.
- **Preview before persist.** Bedrock responses are never stored automatically. The user sees the preview first. Only an explicit Save action writes to the database.
- **Bedrock via VPC Endpoint only.** No public internet path to the AI model. Ever.
- **Secrets Manager for all credentials.** No hardcoded keys, no environment variable shortcuts.
- **AI results stored with full context.** Every saved result is persisted with user ID, template ID, prompt, timestamp, and version — making the audit trail a queryable record, not just a log.

---

## Request Flows

**Normal Application Request**
```
User → Route53 → WAF → ALB → EC2 → Redis(Session Check) → PostgreSQL(Data Read/Write) → EC2 → ALB → User
```

**Login & Template Load**
```
User(Login) → Route53 → WAF → ALB → EC2 → PostgreSQL(Fetch Assigned Template) → Redis(Store Session + Template Context) → ALB → User
```

**AI Request Flow** *(preview only — nothing stored)*
```
User → Route53 → WAF → ALB → EC2 → Redis(Session + Template Context) → DLP Scan → Bedrock(VPC Endpoint) → Labelary API(Generate Preview) → ALB → User
```

**Save Flow** *(only triggered when user clicks Save)*
```
User(Clicks Save) → ALB → EC2 → PostgreSQL(Store Result) → Redis(Cache Update) → ALB → User
```

**Secrets Retrieval**
```
EC2 → Secrets Manager → Retrieve DB/API Credentials
```

**Database Backup**
```
RDS PostgreSQL → Automated Backup → Snapshot Storage → Restore if Needed
```

**Complete End-to-End**
```
Login    → PostgreSQL(Fetch Template) → Redis(Session)
AI Call  → Redis(Session) → DLP Scan → Bedrock(VPC Endpoint) → Labelary(Preview) → User
On Save  → PostgreSQL(Store Result) → Redis(Cache Update)
```

---

## Key Design Decisions Explained

**Why Redis holds the template context**

The assigned ZPL template is loaded into the user's session at login. Every subsequent AI request reads directly from Redis — not PostgreSQL. This avoids a redundant database call on every prompt and keeps AI request latency low.

**Why DLP sits before Bedrock**

A prompt containing injected content or sensitive data should never reach the AI model. Scanning at the EC2 layer before any downstream service is touched ensures the blast radius of a bad input is zero.

**Why Bedrock is accessed via VPC Endpoint**

Routing AI traffic over the public internet introduces latency, exposure, and compliance risk. A VPC Endpoint keeps all Bedrock communication private, within the AWS network, with no IGW dependency.

**Why preview comes before persist**

Bedrock responses are not stored automatically. The user sees the Labelary-rendered preview first and decides. Only an explicit Save action writes to PostgreSQL. This keeps the database clean — only user-approved, meaningful versions are stored. Discarded outputs leave no trace.

**Why PostgreSQL stores the saved result with full metadata**

Every saved ZPL version is stored with user ID, template ID, original prompt, AI response, and timestamp. This is what makes the audit trail real — not a log entry, but a structured, queryable record of every AI-assisted change ever made in the system.

---
