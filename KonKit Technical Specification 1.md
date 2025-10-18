# KonKit Technical Specification - Part I: Architecture, Security, Data & API

**Project:** KonKit  
**Version:** vFinal  
**Status:** ✅ Locked for Design & Development

## 1️⃣ Overview & Objective

This document defines KonKit's **technical foundation** - focusing on architecture, security, data modeling, and API structure. It ensures OWASP Top 10 compliance, scalable system performance, and maintainability.

The system is designed for a content automation platform powered by AI generation pipelines (text, image, and video). The key goals: - Enforce **strong security** across endpoints and user data. - Maintain **scalability** even under high user load. - Allow **resilient, fault-tolerant** operations with background job orchestration.

## 2️⃣ High-Level Architecture

Frontend (Next.js + Tailwind)  
├── Deployed via Vercel (Edge/CDN delivery)  
├── Uses Supabase Auth for login  
└── Calls API Routes (/api/\*)  
<br/>API Layer (Next.js API routes)  
├── Handles requests via serverless functions  
├── Auth middleware verifies JWT tokens  
├── Communicates with Postgres (Supabase)  
├── Publishes jobs to Redis (BullMQ)  
└── Interacts with external AI APIs  
<br/>Worker Layer (Node.js services)  
├── Queues: generate, stitch, moderate, notify  
├── Executes AI orchestration, TTS, FFmpeg tasks  
├── Reports progress via job_executions  
└── Updates results to DB & Library  
<br/>Data Layer (Supabase Postgres)  
├── Stores user data, tasks, assets, and preferences  
└── Indexed & optimized for multi-user queries  
<br/>Storage Layer (Supabase Storage / Cloudflare R2)  
├── Holds generated assets, thumbnails, and videos  
└── Access via signed short-lived URLs  
<br/>Auth & Security  
├── Supabase Auth (JWT-based)  
├── Role-based policies for admin vs user  
└── HTTPS enforced, CSP headers set  
<br/>Monitoring & Observability  
├── Sentry for logs and exceptions  
└── Prometheus/Grafana for system metrics

## 3️⃣ Security Compliance (OWASP-aligned)

### Authentication & Authorization

- JWT-based session via Supabase Auth.
- HttpOnly, SameSite cookies for refresh tokens.
- Access control via middleware verifying user_id claim.
- Role segregation for admin endpoints.

### Input Validation

- Use **Zod** for schema validation at API boundaries.
- Sanitize file names, captions, and descriptions.
- Reject invalid MIME types on upload.

### Injection Prevention

- All DB calls parameterized via Prisma or pg library.
- No dynamic SQL or shell input without sanitation.
- FFmpeg subprocess runs with sandboxed file path.

### Data Protection

- Encrypt secrets in .env files via Vercel/Render Secrets.
- Supabase/Postgres handles data encryption at rest.
- All signed URLs expire after 1-5 minutes.

### Rate Limiting & Abuse Prevention

- 60 requests/minute default limit per IP.
- 10 generations/minute per user.
- Redis-based counters reset every 60 seconds.

### Secure Headers & TLS

- Force HTTPS.
- Add CSP, HSTS, and X-Frame-Options headers.
- Set CORS to restrict allowed origins.

### Logging & Audit

- Log all failed logins, moderation rejections, and failed generations.
- Structured JSON logs, PII redacted.

### Moderation & Content Safety

- Run all media through AI moderation (OpenAI + image classifier).
- Reject explicit/nudity content automatically.
- Log flagged content to moderation_log.

## 4️⃣ Data Model Overview

### Database: Supabase (Postgres)

| Table | Description |
| --- | --- |
| users | Stores user accounts & subscription link |
| content_bases | Defines brand/product context |
| content_assets | Uploaded logos, images, references |
| generation_tasks | Records AI generation jobs |
| generation_variants | Regenerations per task |
| scheduled_jobs | Scheduler configuration per user |
| job_executions | Execution logs for worker tracking |
| credit_wallets | User balances and usage |
| credit_transactions | Ledger for credits used |
| preferences | AI and user default settings |
| moderation_log | Rejected uploads or content |

Indexes: (user_id, created_at), (status, created_at) for each major table.

## 5️⃣ Storage Design

| Type | Service | Description |
| --- | --- | --- |
| **Images & References** | Supabase Storage / R2 | Folder: /user_&lt;id&gt;/bases/&lt;base_id&gt;/assets/ |
| **Generated Videos** | R2 / Worker local temp → R2 | Temporary worker bucket cleared hourly |
| **Previews** | Cached CDN thumbnails | Optimized via Next.js image optimization |

All asset retrieval via **signed URLs** (5-minute expiry).

## 6️⃣ API Design Principles

### Structure

- RESTful JSON endpoints under /api/v1/\*.
- Version control via folder naming.
- All responses use unified envelope:

{ "success": true, "data": {}, "error": null }

### Authentication

- JWT validated per request.
- Middleware extracts user_id.

### Rate Limiting

- Redis keys: rate:&lt;user_id&gt; for counters.
- Returns 429 with retry-after header.

### Idempotency

- All generation endpoints accept idempotency_key header.
- Prevents double-charging or duplicate jobs.

### Pagination

- Cursor-based pagination for large lists (Library, Tasks).

## 7️⃣ Worker & Queue Architecture

| Queue | Function |
| --- | --- |
| generate | Runs AI text + image generation |
| stitch | Combines assets into videos via FFmpeg |
| moderate | Scans outputs for safety |
| store | Uploads completed files to R2 |
| notify | Sends completion notifications |

Workers use **BullMQ** + Redis. Each job updates status in job_executions. Retries: 3 with exponential backoff.

## 8️⃣ Scalability & Performance

### Horizontal Scaling

- Frontend auto-scales via Vercel edge.
- Workers containerized and scaled by queue depth.

### Caching

- Redis cache for common reads (e.g., list of bases).
- Edge caching via Vercel for static routes.

### Connection Pooling

- Use pgBouncer to manage concurrent Postgres connections.

### Load Handling

- Target: 500 concurrent generation jobs.
- Use auto-queue overflow (reject with graceful error) if exceeded.

## 9️⃣ Performance Targets

| Metric | Target |
| --- | --- |
| API latency (non-generation) | ≤200ms |
| Page load (p95) | ≤2.5s |
| Worker task completion | ≤30s for typical generation |
| Database query (p95) | ≤100ms |

## 10️⃣ Cost & Efficiency Controls

- Cache AI responses (short TTL) for regeneration efficiency.
- Use lighter model variants (GPT-4o-mini) for text tasks when possible.
- Async batching for moderation requests.

**Status:** ✅ Technical Spec (Part I) Locked  
**Next Document:** Part II - CI/CD, Deployment, Monitoring, and Development Standards.