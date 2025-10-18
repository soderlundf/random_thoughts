# Production-Readiness Checklist for a Node.js + Express App (with Cron Jobs)

This checklist contains some tips for a **secure**, **stable**, and **robust** application for production environments.

---

## Core Hygiene
- [ ] [Use an **LTS version** of Node.js; pin major/minor (e.g., 22.x LTS).  
  Define `"engines"` in `package.json`.](./core/lts.md)
- [ ] Set **NODE_ENV=production** to disable debugging and detailed stack traces.
- [ ] Use a **lockfile** (`package-lock.json`) and run `npm ci` in CI/CD.
- [ ] Build **immutable, versioned artifacts** (Docker images, bundles).

---

## Express / API Surface
- [ ] **Security headers**: Use `helmet` (HSTS, X-Frame-Options, CSP, etc.).
- [ ] **Rate limiting**: Use `express-rate-limit` for brute-force protection.
- [ ] **Request size limits**: e.g., `express.json({ limit: '1mb' })`.
- [ ] **Timeouts**: Explicitly set `requestTimeout`, `headersTimeout`, and `keepAliveTimeout` to prevent slowloris attacks.
- [ ] **CORS**: Restrict origins, methods, and headers. Enable credentials only if strictly required.
- [ ] **Trust proxy**: `app.set('trust proxy', 1)` when behind a reverse proxy.
- [ ] **Compression**: Only for text, avoid double-compression.
- [ ] **Output escaping**: Sanitize HTML; never use `eval`.

---

## Authentication & Sessions
- [ ] **HTTPS everywhere** (TLS termination at proxy/load balancer).
- [ ] Use **JWTs or session cookies** with short TTL and rotation on logout.
- [ ] **Cookies**: `Secure`, `HttpOnly`, and `SameSite=Lax/Strict`.
- [ ] **CSRF protection** for web forms (e.g., `csurf`).

---

## Input Validation & Sanitization
- [ ] Validate all incoming data using **Zod/Joi/Yup** schemas.
- [ ] Enforce type checks and defaults for both request and response payloads.
- [ ] For file uploads:
  - Check file type and size.
  - Sanitize filenames.
  - Store outside webroot.
  - Stream to disk/cloud securely.

---

## Error Handling & Robustness
- [ ] Centralized **error middleware** that logs but never exposes stack traces.
- [ ] Consistent JSON error structure with HTTP status + code + message.
- [ ] **Timeouts, retries, and backoff with jitter** for external calls.
- [ ] **Circuit breakers** (e.g., `opossum`) to prevent cascading failures.
- [ ] Make **idempotent** write endpoints (e.g., via idempotency keys).

---

## Cron Jobs
- [ ] Run cron jobs in a **separate process/container** from the web app.
- [ ] Ensure **singleton execution** (distributed locks or job queue).
- [ ] Prevent **overlaps** (“skip if running” or serialize queue).
- [ ] Make jobs **idempotent** (safe to rerun).
- [ ] Use **retry with exponential backoff** and a **dead-letter queue** for failures.
- [ ] Log job executions (start, end, duration, result, errors).
- [ ] Define **catch-up policy** (skip or backfill missed runs).
- [ ] Implement **graceful shutdown**: stop new jobs, await in-progress with timeout.
- [ ] Use **UTC scheduling** internally to avoid DST issues.
- [ ] Expose metrics per job: run count, failures, duration histograms.

---

## Observability
- [ ] Structured **JSON logging** (e.g., `log4js`, `pino`) with correlation IDs.
- [ ] Mask sensitive data in logs.
- [ ] Expose **/live** (liveness) and **/ready** (readiness) endpoints.
- [ ] Export **Prometheus/OpenTelemetry metrics**:
  - Request latency (p50/p95)
  - Error rates
  - Memory, CPU, event loop lag
  - Cron job metrics
- [ ] Alerts on SLO breaches (latency, error %, job failures, memory leaks).

---

## Dependencies & Supply Chain
- [ ] Regular `npm audit` or `snyk` scans.
- [ ] Review transitive dependencies; avoid unmaintained packages.
- [ ] Freeze critical dependency versions; use lockfile updates via CI.
- [ ] Generate an **SBOM** (e.g., with Syft).
- [ ] Enable **secret scanning** and license compliance checks.

---

## Configuration & Secrets
- [ ] Follow **12-factor app** principles: all config via environment variables.
- [ ] Store secrets securely (Vault, Docker secrets, K8s secrets).
- [ ] Never log secrets.
- [ ] Use **feature flags** for safe deploy toggles.

---

## Databases & External Services
- [ ] **TLS** for DB connections.
- [ ] Configure **connection pooling** (min/max).
- [ ] **Schema migrations**: backward-compatible and versioned.
- [ ] Maintain **backups** and regularly test restores (RPO/RTO).

---

## Containers (Docker/Kubernetes)
- [ ] Use **minimal base images** (`-alpine` or distroless).
- [ ] Run as **non-root**; use `USER node`.
- [ ] Set **read-only rootfs** and minimal capabilities.
- [ ] Include a `HEALTHCHECK` (calls `/live` or `/ready`).
- [ ] Define **resource limits** (CPU/mem) and handle signals (e.g., with `tini`).
- [ ] Configure restart policy — but don’t rely on it as error handling.

---

## Networking & Scaling
- [ ] Use **reverse proxy** (Nginx/Traefik) for TLS, HTTP/2, header sanitation.
- [ ] Keep web app **stateless**; use Redis or DB for session store.
- [ ] Set **client timeouts** and **connection reuse** for outbound requests.
- [ ] Define **service-level budgets (SLOs)** per component.

---

## Testing & Quality
- [ ] **Unit + integration tests**, including edge cases and failure modes.
- [ ] **E2E tests** against staging with realistic data.
- [ ] **Load tests** with representative traffic.
- [ ] **Fuzzing** for parsers and input endpoints.
- [ ] Benchmark hot paths (CPU, memory, event loop lag).
- [ ] Align with **OWASP ASVS** security controls.

---

## Deployment & Operations
- [ ] Use **blue/green or canary** deployments with rollback capability.
- [ ] Apply **migrations before code rollout**; remove deprecated fields last.
- [ ] Maintain a clear **CHANGELOG** and **runbooks** for incident handling.
- [ ] Enforce **log and metrics retention** (and privacy rules).

---

## Data Protection & Access Control
- [ ] Minimize **PII** storage; pseudonymize when possible.
- [ ] Enforce **RBAC** for admin endpoints.
- [ ] Keep **audit logs** for administrative actions.
- [ ] Apply **GDPR-compliant retention and deletion policies**.

---

## Common Gotchas
- [ ] Missing request timeouts → hung sockets, resource leaks.
- [ ] Multiple cron runners after scaling → use distributed locking.
- [ ] DST/timezone bugs → schedule in UTC.
- [ ] Sensitive data accidentally logged → mask and audit log output.
- [ ] Default Node timeouts left undefined → set explicitly.
- [ ] Forgot `trust proxy` → wrong IPs, broken rate limiting.

---
