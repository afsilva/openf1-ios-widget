# Security assessment — OWASP Top 10 (2021)

Scope: `openf1-ios-widget` active iOS-only tree (`XcodeProjectTemplate/*`).

Assessment result: **PASS (with medium-priority hardening recommendations)**

Audit date: 2026-05-31

---

## A01: Broken Access Control — PASS
- App is read-only against public OpenF1 endpoints.
- No login/session/role model and no privileged local operations.

## A02: Cryptographic Failures — PASS
- API base is fixed HTTPS: `https://api.openf1.org/v1`.
- No API secrets/tokens are handled in source or cache.
- No plaintext credential storage patterns observed.

## A03: Injection — PASS
- Endpoint allowlist enforced (`meetings`, `sessions`, `session_result`, `drivers`).
- Query strings validated against a restrictive regex before network call.
- Rendered UI strings are sanitized (control-char stripping + length bounds).

## A04: Insecure Design — PASS
- Cache-first with bounded data + graceful fallback/last-known-good behavior.
- Failures degrade to safe, non-crashing UI states.
- Refresh cadence is controlled (daily / hourly around weekend).

## A05: Security Misconfiguration — PASS (with recommendations)
- Network timeout set explicitly.
- Response and cache sizes bounded.
- App-group storage used with fallback to sandbox caches.
- **Recommendation:** keep App Group identifier configurable per deploy environment (no personal identifiers in tracked config).

## A06: Vulnerable and Outdated Components — PASS (process dependent)
- Core stack uses Apple frameworks (SwiftUI/WidgetKit/AppIntents/Foundation).
- No third-party package manager dependencies detected.
- Security status depends on keeping Xcode + iOS SDK current.

## A07: Identification and Authentication Failures — PASS (N/A)
- No authentication flows in current architecture.

## A08: Software and Data Integrity Failures — PASS
- Strongly typed decode (`Codable`) throughout API payload handling.
- No dynamic code loading/eval patterns.
- Cache persistence guarded by size constraints and bounded endpoint count.

## A09: Security Logging and Monitoring Failures — PARTIAL / LOW RISK
- User-facing failures are genericized (good).
- No structured production telemetry or security event stream is implemented.
- **Recommendation:** add optional structured app logging policy for release builds (privacy-safe, no PII/secrets).

## A10: SSRF — PASS
- Remote host is fixed and non-user-controlled.
- Endpoint path is allowlisted; arbitrary host redirection path not exposed.

---

## Additional controls verified
- Response body cap (`maxResponseBytes = 1 MB`)
- Cache file cap (`maxCacheBytes = 2 MB`)
- Endpoint cache entry cap (`maxEndpointEntries = 200`)
- Sanitization before render for driver/team/location/title text
- Stale-cache rescue for schedule/results/driver directory endpoints
- Last-known-good standings/calendar fallback to reduce empty-state regressions

## Findings summary

### No critical/high findings
No exploitable patterns corresponding to critical/high OWASP Top 10 violations were identified in current active code.

### Medium-priority hardening opportunities
1. **Transport hardening:** consider certificate/public-key pinning strategy (operationally optional for this app, but improves MITM resilience if threat model requires it).
2. **Observability:** add structured release logging for refresh failures/rate limits with privacy-safe fields only.
3. **Config hygiene:** keep signing/team identifiers and environment-specific values out of tracked files (already enforced; continue policy).

### Low-priority notes
- `isValidPathAndQuery` uses a strict regex; keep this validation in sync if query shapes expand.
- Reassess all OWASP categories if auth/account features are ever introduced.

---

## Recommended next actions
1. Keep current secure-by-default API access model (fixed host + allowlist + bounds).
2. Add a lightweight release logging guideline (what is logged, retention, redaction).
3. Add CI checks for:
   - accidental signing identifier commits,
   - secrets/cert/provisioning files,
   - version/build metadata policy (`YYYY.MM.DD` + incremental build).
