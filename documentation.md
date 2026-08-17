# Project Documentation

Developer-facing reference for iBuild: structure, stack, dependencies, and how
the files relate. If you are an AI or a developer joining this project, this is
the map. For project state and decisions, read
[code_of_conduct.md](code_of_conduct.md).

## 1. Purpose

A collection of instruction files that any AI tool reads before building. The
files encode rules, taste, and state so the AI does not have to re-derive them
from the whole codebase or from chat history.

## 2. The collection structure

```
iBuild/
├── AGENTS.md                  index and rule hierarchy
├── README.md                  user-facing overview
├── documentation.md           this file, developer context
├── code_of_conduct.md         live project state, updated before each commit
├── security.md                audit runbook and audit state
├── skills/
│   ├── frontend-design.md     visual authority
│   └── backend.md             server/data authority
```

Roles:

- **AGENTS.md** is the single source of truth for agent behavior. It defines
  the instruction hierarchy, the domain authority map, and the non-negotiable
  rules. New skills are registered here.
- **code_of_conduct.md** is the state graph. An AI serializes what changed and
  what was decided before every commit, so a fresh session resumes cleanly.
- **security.md** defines audits (SEO, vulnerabilities, quota) and records
  where the last audit stopped.
- **skills/** holds one prescriptive file per domain. Each is a decision
  engine, not a suggestion list.

## 3. Stack

The stack is chosen by the skills, not hardcoded assumptions:

- **Frontend**: [skills/frontend-design.md](skills/frontend-design.md)
  drives all visual, typographic, and interaction decisions.
- **Backend**: [skills/backend.md](skills/backend.md) assumes **Firebase**
  (Firestore + Firebase Auth) as the default, **MongoDB** (self-hosted free
  server + Mongoose) as the alternative, and **Supabase** (Postgres + RLS)
  only rarely. Free-tier-first: no Cloud/Edge Functions, client-direct
  reads/writes, rules/RLS as the enforcement layer.

## 4. The app: Next.js case study

The default shape of an app built from this collection: **Next.js (App
Router)**, Tailwind CSS, plain **JavaScript (`.js`/`.jsx`)**, no TypeScript.
When asked to create a fullstack app, scaffold this structure first, then
customize.

### 4.1 The full app structure

```
project/
├── jsconfig.json                path aliases (@/ -> ./)
├── next.config.js
├── .env.local                   local env config (never committed)
├── constants.js                 menu items, route lists (single source)
├── lib/
│   ├── firebase.js              client init (or the stack's db client)
│   ├── auth.js                  auth init, session helpers
│   └── models/                  per-collection data access: posts.js, users.js
│                               (add models here as the schema grows)
├── contexts/
│   └── auth-context.jsx         AuthProvider + useAuth hook
├── components/
│   ├── layout/
│   │   ├── navbar.jsx           top bar: desktop menu + mobile hamburger
│   │   ├── mobile-menu.jsx      hamburger drawer (mobile)
│   │   ├── sidebar.jsx          logged-in desktop nav
│   │   └── bottom-nav.jsx       logged-in mobile bottom nav
│   ├── ui/                      button, input, card, modal, spinner...
│   ├── auth/                    login-form, signup-form
│   └── post/                    post-card, post-form
├── app/
│   ├── layout.jsx               root: fonts, providers, Navbar
│   ├── page.jsx                 public landing
│   ├── (auth)/
│   │   ├── layout.jsx           centered, no Navbar
│   │   ├── login/page.jsx
│   │   └── signup/page.jsx
│   ├── (app)/                   logged-in, protected
│   │   ├── layout.jsx           auth guard + Sidebar + BottomNav
│   │   ├── dashboard/page.jsx
│   │   ├── profile/page.jsx
│   │   └── settings/page.jsx
│   └── api/                     route handlers (see 4.4)
│       ├── auth/login/route.js
│       ├── auth/signup/route.js
│       ├── auth/logout/route.js
│       ├── user/me/route.js
│       ├── user/[id]/route.js
│       ├── posts/route.js
│       ├── posts/[id]/route.js
│       └── health/route.js
```

What each part is for:

- **Root config**: `jsconfig.json` sets the `@/` alias so imports read
  `@/components/ui/button` instead of long relative paths. `.env.local` holds
  local secrets and is never committed. `constants.js` is the single source of
  truth for nav menus and route lists.
- **`lib/`**: non-component code. `firebase.js` initializes the client once
  (Firebase, or the chosen stack's db client). `auth.js` is the auth init and
  session helpers. `models/` holds one data-access file per collection
  (`posts.js`, `users.js`); add a file when a new collection appears. No data
  access lives in components.
- **`contexts/`**: React context providers. `auth-context.jsx` is the only
  required one; it owns the user's auth state app-wide.
- **`components/`**: everything visual. `layout/` is the navigation shell
  (Navbar, hamburger drawer, Sidebar, BottomNav). `ui/` is shared primitives.
  `auth/` and `post/` are feature components; new features get their own
  folder here.
- **`app/`**: Next.js routes. Root `layout.jsx` renders fonts, providers, and
  the Navbar for everything. `(auth)/` is the public, login/signup area
  without the app shell. `(app)/` is the protected area: its layout guards
  auth and renders the logged-in shell (Sidebar + BottomNav). `api/` holds
  all route handlers. Route groups `(auth)` and `(app)` affect layout only,
  not the URL.

### 4.2 Auth context

`contexts/auth-context.jsx` exports `AuthProvider` (subscribes to auth state
once at startup, exposes `{ user, loading, login, logout, signup }`) and
`useAuth()`, the only way components read the user. `app/layout.jsx` wraps the
tree with `AuthProvider`. The `(app)/layout.jsx` guard reads `useAuth()`:
spinner while loading, redirect to `/login` when logged out, otherwise render
the logged-in shell.

### 4.3 Navigation

- **Public**: Navbar at top. Desktop shows a horizontal menu (Home, About,
  Pricing, Login). Mobile collapses into a hamburger button opening
  `mobile-menu.jsx`; closes on link click, outside click, Escape.
- **Logged in**: Navbar slims to brand + user menu. Desktop gets a persistent
  **Sidebar** (Dashboard, Profile, Settings). Mobile gets a fixed **bottom nav
  bar** instead; Sidebar hidden on mobile, bottom nav hidden on desktop.
- Menu items live once in `constants.js` (a small file at project root),
  consumed by Navbar, Sidebar, and BottomNav. Active route is highlighted from
  the current path.

### 4.4 API routes (dummy contract)

Next.js route handlers, each returning the error contract
`{ error: { code, message } }` on failure.

| Route                          | Methods              | Purpose                                   |
| ------------------------------ | -------------------- | ----------------------------------------- |
| `/api/auth/login`              | POST                 | validate credentials, set session         |
| `/api/auth/signup`             | POST                 | create user, set session                  |
| `/api/auth/logout`             | POST                 | clear session                             |
| `/api/user/me`                 | GET                  | current user profile                      |
| `/api/user/[id]`               | GET                  | public profile by id                      |
| `/api/posts`                   | GET / POST           | paginated list / create                   |
| `/api/posts/[id]`              | GET / PATCH / DELETE | single post                               |
| `/api/health`                  | GET                  | liveness, returns ok                      |

Rules: validate at the boundary (zod), honest status codes, paginate lists
(`?page=1&pageSize=10`), no secrets or internals leaked, and minimize quota
per [skills/backend.md](skills/backend.md).

## 5. How to use: setup the app with an AI

Ask the AI to scaffold the project. The exact prompt is:

> Set up a fullstack Next.js app for me using the folder structure in
> [documentation.md](documentation.md). JavaScript only (`.js`/`.jsx`), no
> TypeScript. Create every folder and file in the tree in section 4.1,
> install the dependencies in section 6, and
> make every page and API route render with dummy data so the app runs end to
> end. Follow the auth context, navigation, and API contract sections. Use
> the design rules in [skills/frontend-design.md](skills/frontend-design.md).
> Then stop and let me customize.

Rules for the setup:

- The AI creates the whole tree, no files left out, even if a file is a stub.
- Dependencies come from section 5 and only what the app needs; nothing extra.
- Pages and routes return dummy data until real backend wiring is added later.
- Design follows the skill from the first screen, not default styling.
- After setup the AI stops; it does not start building features on its own.

## 6. Dependencies

Prefer what is already installed. Never assume a library exists; check the
project files first. Base toolchain for the Next.js case study:

- **Core**: `next`, `react`, `react-dom`, `tailwindcss` (or the
  project's existing styling setup), `clsx` + `tailwind-merge` for
  class utilities.
- **Validation**: `zod` for boundaries and schema mirroring.
- **Per backend stack**:
  - **Firebase**: `firebase` (auth + firestore client). Emulator for rules
    tests.
  - **MongoDB**: `mongoose` + `express`/`fastify` (unless running inside
    Next.js API routes, then Next only), `bcrypt`/`argon2`, `cookie`/`jose`
    or sessions for auth.
  - **Supabase**: `@supabase/supabase-js` (auth + client). SQL migrations in
    the repo. No extra ORM needed.
- **Env**: `dotenv` or the platform's env config.

Verify against the skills before installing. Never install a dependency just
because it exists; the feature must need it.

## 7. Working rules

- Read **AGENTS.md** first, then the domain skill(s) for the task, then
  **code_of_conduct.md** for current state.
- Before a commit: update **code_of_conduct.md** with what changed and any
  decisions made. Keep entries short.
- On an audit request: follow **security.md**, then update its state section.
- Add new domains as `skills/<domain>.md` and register them in AGENTS.md.
- No em dashes in any file in this collection. Write short, concrete,
  opinionated sentences.