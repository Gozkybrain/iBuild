# Security

Audit runbook and audit state. This file is an instruction: when an audit is
requested, follow it exactly, then update its state so the next audit resumes
where this one stopped.

## 1. How to trigger an audit

Ask the AI. The exact prompt is:

> Run an audit on this project following [security.md](security.md). Work
> through every applicable item in section 2, report findings in the format
> in section 3, then record the outcome in the audit state so the next audit
> resumes where this one stopped.

Do not improvise past this file. The checklist below is the whole scope of an
audit.

## 2. How to run an audit

1. Read **AGENTS.md**, this file, and **code_of_conduct.md**.
2. Work through the checklist below. Cover every applicable item.
3. Record results in **section 5 (Audit state)**: what passed, what failed,
   where you stopped, next steps.
4. Do not claim a clean audit if steps were skipped. Say what was not covered.

## 3. Audit checklist

### 3.1 Secrets and config

- Search the repo for keys, tokens, passwords, connection strings, and
  `.env` files. Any found: flag as critical, remove, rotate.
- Firebase config and Supabase anon keys are public by design. Service account
  keys and `service_role` keys must never be in client bundles or the repo.
- Environment values come from config, never hardcoded.

### 3.2 Vulnerabilities

- Dependency scan: flag outdated or known-vulnerable packages, pin or bump them.
- Injection: no raw SQL/string-built queries touching user input without
  parameterization. Mongo: no operator injection in `$` fields; reject them.
- Mass assignment: unexpected fields are stripped or rejected on writes.
- Broken access control: every protected path enforces authorization at the
  data layer (rules/RLS/server middleware), not by hiding UI.
- Auth hygiene: hashed passwords (bcrypt/argon2), expiring sessions/tokens,
  no credentials logged or echoed.
- Rate limits on auth paths (Firebase reCAPTCHA/App Check, Supabase auth
  limits, middleware on the Mongo path).

### 3.3 Security rules / RLS

- Firestore rules: default deny, owner-scoped, reads/writes split, writes
  validated in rules. Storage rules get the same treatment.
- Supabase: RLS enabled on every table, policies per access pattern.
- Rule tests run and pass.

### 3.4 SEO and web hygiene

- Meta tags, title, description, canonical, Open Graph where applicable.
- Headings hierarchy sane; one `h1` per page.
- Semantic HTML, descriptive alt text, tiny-text and tap-target audits.
- `robots.txt` and `sitemap.xml` present, correct for the environment.
- Lighthouse basics: CLS, LCP, INP reasonable. Lazy-load below-fold media.
- No blocked content (invisible text, cloaking, doorway pages).

### 3.5 Performance and quota posture

- Read/write per screen minimized per [skills/backend.md](skills/backend.md)
  (no N+1, paginated lists, field masking, one doc per screen).
- No client retry loops that can spin reads; backoff on failures.
- No unbounded caches without invalidation.
- Storage: nothing written that the feature does not need.

### 3.6 Frontend review (whatever the app looks like)

- Visual decisions match [skills/frontend-design.md](skills/frontend-design.md);
  no AI-aesthetic leaks.
- No secrets or infra details reach the client bundle.

## 4. Expected output

The audit report, delivered in the chat and summarized into the audit state.
Every finding is a numbered list item, each with:

- **Severity**: critical / high / medium / info.
- **Location**: file and line or feature.
- **What is wrong**, in one or two sentences.
- **The fix**, in one or two sentences.

The report closes with a short verdict:

- A list of anything **not covered** (sections skipped, reasons).
- The **next audit focus**: the highest-severity open items or the skipped
  area, so the following audit resumes there.

Then the state section below is updated with the same information. The report
is for the user; the state section is the record for the next audit.

## 5. Audit state

Last audit date:

Nothing audited yet. Next audit: on first request, run section 3 fully and
record the outcome here. Keep the newest entry at the top.

- **2026-08-17**: baseline created. No audit run. Awaiting first request.