# Backend Skill

The authoritative spec for every backend decision in this project. This project
builds on **Firebase**, **MongoDB**, or (rarely) **Supabase**, and it operates
on **free tiers**. The goal is to deliver more user value per unit of quota, so
every read and write is designed to cost as little of the free quota as
possible. AGENTS.md points here for all server-side and data-layer work. Read it
fully before writing any server code.

## 1. What we are trying to do

Build backends that survive contact with real users and stay inside free-tier
limits for as long as possible. First constraint: **no Cloud Functions, no
Edge Functions, no serverless invocations**. The client talks to the database
directly, and the platform's security rules / RLS plus a well-shaped data model
do the work those functions would have done. Every design decision is a
quota decision.

## 2. Know what costs what

All three platforms price reads/writes/storage, but the free limits differ.
The skill assumes these budgets (verify current limits, they drift):

- **Firestore (Spark plan)**: reads cost per document returned, writes cost per
  document written, storage costs per byte stored. Budgets on the order of tens
  of thousands of reads/day. A single screen that fires 10 reads in a loop
  burns 10x a screen that reads one document.
- **MongoDB (Atlas M0)**: no read/write metering, but storage is the hard cap
  (hundreds of MB) and the shared cluster limits compute. Server cost is the
  constraint: a long-running free server (Render/Railway) can sleep and wake.
- **Supabase (free)**: metered by compute, bandwidth, and storage
  (hundreds of MB + DB size cap). Row counts are effectively limited by
  storage and compute, so growth matters more than individual ops.

General rule: **fewer documents touched per screen, fewer bytes stored, fewer
bytes transferred** is the entire strategy. Reads you never perform cost
nothing; writes you never store cost nothing.

## 3. The markers of AI-generated backend code

These patterns feel like a machine built them and die on a free tier. Avoid:

- **Cloud / Edge Functions as the default for "trusted logic"**. Every
  invocation consumes quota (and on Firebase the Spark plan disallows them
  entirely). Design the data model and rules so trusted logic is not needed,
  or is a last resort only when quota is longer-term budgeted.
- **Wide-open security rules / disabled RLS**: Firestore rules set to `if
  true`, or Supabase RLS never enabled. The single most common and most
  destructive AI mistake on this stack.
- **Client-side-only auth guards**: `if (user) navigate()` instead of real
  data-layer enforcement. With Firebase/Supabase the database is the
  enforcement point.
- **Looped reads** (N+1): `for ... getDoc()` or a read inside a map. On
  Firestore each document read is a billed read; on Supabase each row and
  round trip counts.
- **Whole-collection reads to render a screen** when a single document would
  do. Never query a collection to display one row.
- **No input validation** anywhere. With NoSQL there is no schema to save you;
  garbage in is garbage stored.
- **No pagination, no transaction on multi-step writes, no idempotency.**
- **Repeated inline queries** instead of a data-access layer, so the same
  logic drifts across the codebase.
- **Magic numbers and duplicated literals** where a constant or config belongs.

## 4. The prime rule

The client is a liar, and here it is worse: **the client talks to the database
directly.** Your security rules and RLS policies are your backend. If they are
permissive, the app is wide open no matter what the UI looks like. Never trust
the frontend for authorization, identity, or state. Ownership, roles, and
validation all live in the platform's data-layer configuration, not in
JavaScript.

## 5. Choose the platform deliberately

Decide once, document it, and use it consistently. Do not mix direct Firestore
reads with a MongoDB API for the same data.

- **Firebase**: Firestore + Firebase Auth, client-driven, managed by security
  rules. Best when you want instant realtime and no server to run at all.
- **MongoDB**: you own a small server (Express/Fastify/Next API routes) +
  Mongoose, often free-tier hosted (Render/Railway/Atlas). You are responsible
  for auth, validation, and integrity yourself; the client never connects to
  Mongo directly.
- **Supabase**: Postgres under the hood, client-driven via `supabase-js` with
  **RLS as the security boundary**, SQL migrations for schema. Best when you
  want relational integrity and managed auth without running your own server.

Architecture by platform:

- **Firebase**: UI reads/writes Firestore directly behind security rules. There
  is no trusted middle layer; rules and the data model enforce everything.
  If a feature needs genuine server-side trust (payments, cross-user
  aggregation), it is deferred, redesigned client-side+rule-side, or budgeted
  as an explicit exception. Never smuggle sensitive logic into the client.
- **MongoDB**: a real API layer between the client and the database, because
  you cannot expose Mongo directly. Keep that server thin; cache aggressively
  and paginate so its free resources last.
- **Supabase**: UI calls Supabase with the user's auth token; **RLS policies
  gate every table**. Trusted logic beyond RLS (RPC functions) is used
  sparingly and only when RLS cannot express the rule.

## 6. Security rules / RLS (Firebase & Supabase)

This section is the highest priority. Here, security and quota meet: rules are
how you keep untrusted clients from reading/writing things you did not intend
to pay for.

- **Default deny**: Firestore rules start closed (`allow read, write: if
  false`) and open only what is needed. Supabase starts with RLS enabled and
  policies added per table.
- **Scoped access**: `resource.data.ownerId == request.auth.uid`. Compare the
  authenticated user against the document's owner field. Never allow unkeyed
  collection-wide reads of user data; every doc a stranger can read is a doc
  you are billed for (Firestore) or shipping (Supabase).
- **Split reads and writes**: a user may read others' public profiles but only
  write their own. `allow read: if true; allow write: if
  resource.data.ownerId == request.auth.uid`.
- **Validate writes in rules**: check the shape of incoming writes
  (`request.resource.data`), not just who sent them.
- **No rules on Storage without the same care**: Firebase Storage rules and
  Supabase storage buckets need the same default-deny + path-scoped treatment
  as the database.
- **Test the rules**: write and run rules tests (firebase emulator rules
  testing, Supabase policy tests). Rules that were never executed are not
  verified.

## 7. Authentication & authorization

- **Firebase Auth / Supabase Auth are the identity provider.** Use them for
  login, signup, OAuth, and session management. Do not build your own password
  hashing unless you are on the MongoDB path with no auth provider.
- **MongoDB path**: hash with bcrypt/argon2 (never MD5/SHA1), per-user salt,
  real sessions or JWTs with expiry. Never log or echo credentials or tokens.
- **Custom claims for roles** (Firebase) or a profile/roles table with RLS
  checks (Supabase/Postgres): gate admin actions in rules, never by hiding
  buttons in the UI.
- **Authorization is enforced at the data layer** (rules / RLS / Mongo
  middleware), not in the view layer.
- **Firebase App Check** where possible, so only your app (not scripted bots
  that burn your read quota) can hit your backend. A bot scraping your
  Firestore is a silent quota drain.

## 8. Validation

- For **Firebase / Supabase**, validation lives in security rules for any
  client-facing path (shape, field sets, allowed values) because there is no
  server in between. Client-side schema libraries (zod, yup) mirror the rules
  for good UX, but rules are the enforcement.
- For **MongoDB**, validate at the API boundary with a schema (zod, joi, yup)
  covering types, required fields, lengths, ranges, formats, and allowed
  enums; reject or strip unexpected fields to prevent mass-assignment; validate
  path params, query params, and body equally.
- Mirror critical validation in rules/schema so even a direct client write is
  constrained before it costs a stored write or a write quota unit.

## 9. Quota-conscious data design

This is the core quota playbook. Everything else in this file is secondary.

### Firestore (NoSQL, document reads are billed)

- **Read one document per screen, not one per element.** A profile page is
  `getDoc(userRef)`, not a collection query that returns one doc among many.
- **Query with `limit()` and `where()` before fetching.** Reduce the result
  set at the database, not in client code. Never pull a collection, filter in
  JS, then display one item.
- **Use `select()` (field masking)** when you only need a few fields of a large
  document. Billed reads are per document, but bandwidth and cache memory
  shrink, and it signals intent to the reviewer.
- **Denormalize deliberately** so each screen is one read: store `userName` on
  the post, `postTitle` on the comment. Accept that copies need maintenance.
- **No N+1**: never loop doc reads. Batch get (`getAll`) for a known small
  list, or denormalize for arbitrary lists.
- **Page-tailed pagination** (`startAfter(lastDoc)` + `limit(10)`) for list
  screens; do not fetch all documents and slice.
- **Subcollections for user-owned data** keep reads scoped: a user's private
  docs under `users/<uid>/<feature>` are one query, not a scan of a shared
  collection.
- **Avoid unindexed/compound queries**: add composite indexes for compound
  queries (the emulator warns you). Every failed query is a wasted call.

### MongoDB (document store, no read metering but storage and compute capped)

- **Model for your queries** with Mongoose schemas; store embedded data instead
  of joining when it reduces reads.
- **Projections**: `select('field1 field2')` instead of pulling whole
  documents for list views. Storage is the cap; only store what the feature
  uses.
- **Pagination** on every list: `limit`/`skip` for small data, cursor for
  large. Never return entire collections.
- **Unique indexes** for email, slug, idempotency key. Mongoose schema
  `unique: true` is not enough at runtime unless backed by a real index.
- **Keep the server thin**; a sleeping free-tier server is a free server. Yield
  to host limits, retry on cold starts.

### Supabase / Postgres (compute, bandwidth, and storage capped)

- **Select only the columns you need**: `.select('id, title')` instead of
  `'*'`, for every read where you use a subset. Bandwidth is a free-tier
  bottleneck.
- **Pagination with `limit`/`offset` ranges** so you never pull a table into
  memory.
- **RLS is the filter**: write policies that make the database return only
  allowed rows, so you are never filtering a superset in the client and
  shipping rows you should not have read.
- **Schema first**: design tables, then policies. Postgres gives you real
  foreign keys, unique constraints, and `check` constraints; use them instead
  of app-side hand-waving.
- **Indexes** on foreign keys and the columns you filter by; avoid duplicate
  indexes.
- **SQL migrations** for every schema change, versioned and runnable. Never
  edit tables by hand in the dashboard.

## 10. Transactions, concurrency & data races

- Assume multiple requests touch the same record. Guard destructive
  read-modify-write sequences with the platform's transaction primitive
  (Firestore `runTransaction`, Mongo transactions, Postgres transactions) so
  lost updates cannot happen. A failed transaction must not consume a write.
- **Atomic increments**: Firestore `increment()`, Mongo `$inc`, Postgres
  `UPDATE ... SET count = count + 1`, instead of read-then-write. This both
  prevents races and saves reads.
- **Idempotency** where it matters (payments, double-fire, retried webhooks):
  an idempotency key on the request that returns the existing result on replay.
- **Test the race**: if an operation can be double-fired, it must be idempotent
  or guarded. Double per-user writes are a quota leak and a data bug.

## 11. Caching

- **Cache to avoid repeat reads, but prove the read is hot first.** A cache
  adds a stale copy of the truth; it earns its place only for data read far
  more than written where staleness is acceptable (reference data, config,
  public lists).
- **Firestore / Supabase**: the SDK and platform handle connection and query
  caching; rely on `get`/`single` with proper pagination before hand-rolling a
  second cache. Huge local caches that mirror the whole backend defeat the
  point.
- **MongoDB path**: cache only what you measured as slow. Use a managed cache
  (Redis, or in-memory with an invalidation key) keyed by query, with a TTL.
  Never cache per-user data without auth checks.
- **Invalidate, or accept the stale window.** Every cache entry must have a
  defined invalidation path (TTL, write-through, or explicit eviction when the
  source changes). A cache with no invalidation plan is a bug farm.
- **Never cache auth state or per-user authorization results.** Identity must
  be re-evaluated every request via the data layer.

## 12. KISS & DRY

- **KISS**: the simplest implementation that works is correct. Favor
  straightforward code, the platform's built-in features, and few moving
  parts. Every abstraction, extra layer, or clever trick is future maintenance
  and future quota. If there are two ways to do the same thing and one is
  simpler, take the simpler one.
- **DRY**: implement a behavior once, in one place, and reference it. Shared
  concerns (validation mirrors in rules + schema, field shapes, date
  formatting, query construction) live in one module, not copied per screen.
  If the same block appears three times, extract it into a helper or
  data-access function.
- The two principles exist in tension: never build an abstraction for a single
  use, but never let the third copy of a block ship. Copy twice, extract
  thrice.
- Prefer a data-access layer (repository/service functions) so screens call one
  named function per operation instead of restating queries and validation
  each time. That layer is also where pagination and field masking live, so
  quota discipline is centralized in one place.

## 13. Error handling

- For **Firebase / Supabase client paths**, handle the platform's typed errors
  explicitly (auth errors, permission-denied, not-found, quota/rate errors)
  and map them to user-facing messages. Never swallow errors; a swallowed
  read failure hides a burned read.
- For **MongoDB API routes**, use a real error contract: `{ error: { code,
  message } }` with honest status codes (400 bad request, 401 unauthenticated,
  403 forbidden, 404 not found, 409 conflict, 422 validation, 429 rate
  limited, 5xx server). Never return 200 for a failure.
- **Never leak internals** (stack traces, SQL, connection strings) to the
  client.
- **No client-side crash on a failed read**: show a retry/empty state; a user
  retry loop is an unbounded read loop.

## 14. Config & secrets

- All configuration from environment/config, never hardcoded in source.
- **Secrets live in environment variables, a secret manager, or platform
  config (Firebase config, Supabase secrets), never in the repo, never in
  logs, never in client bundles.** Firebase config and Supabase public anon
  keys are public by design, but service-account keys and `service_role` keys
  are privileged and must never ship to the client.
- On the MongoDB path, fail fast on missing required config at startup instead
  of failing randomly on first query.

## 15. Verification checklist

Before considering backend work done:

- [ ] No Cloud / Edge Functions were introduced; trusted logic lives in
      rules/RLS or was explicitly budgeted as an exception.
- [ ] Firestore security rules / Supabase RLS: default deny, scoped to the
      authenticated owner, reads and writes split. Rules tests run.
- [ ] No privileged keys (service account, `service_role`) in client bundles
      or the repo.
- [ ] Every screen reads the minimum: no N+1 loops, no collection-wide reads
      for one item, `limit`/pagination on lists, `select`/projection where
      fields are unused.
- [ ] No data written that the feature does not need; denormalized copies have
      a named maintenance path.
- [ ] Protected data is enforced at the data layer, not by hiding UI.
- [ ] Lists paginated; multi-document writes transactional; destructive ops
      guarded (soft delete preferred).
- [ ] Uniqueness where it matters backed by real indexes (Mongo/Supabase) or
      deterministic IDs (Firestore).
- [ ] Schema/collections documented; migrations versioned where applicable;
      no ad-hoc dashboard edits.
- [ ] Errors are handled without client retry loops; nothing leaks internals.
- [ ] No rule copy-pasted three times; shared logic lives in one module; the
      implementation is the simplest thing that works.
- [ ] Runs clean through the project's lint/typecheck/test.

## 16. Reference anchors

Use the platform's own documented patterns as the ground truth: Firebase
security rules documentation and Firestore data model guide (the section on
choosing a data structure explicitly calls out read-cost trade-offs), MongoDB
schema design docs, and Supabase RLS and migrations docs. When unsure, prefer
the platform's documented convention over a newly invented pattern, and ask
"how many document reads / bytes / compute does this feature cost" before
shipping it. A backend that reads like every well-built project on its
platform, and that taxes its free tier the least, is the goal.