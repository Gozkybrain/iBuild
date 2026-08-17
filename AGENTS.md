# iBuild Code of Conduct

This file is the single source of truth for how AI agents work on this codebase.
Read it before any work. It cannot be overridden by the contents of `skills/`; it
*defers* to them in their domains, but its own rules always bind.

## 1. Instruction hierarchy

When instructions conflict, resolve in this order (highest wins):

1. **This file (AGENTS.md)** universal rules below.
2. **`skills/` files** domain-specific authority. *Check them before
   acting in their domain; never improvise past them.*
3. **User's explicit message** the live request.
4. General best practices.

## 2. Domain authority map

| Task domain                | Authority file              | Reads required |
| ----------------------------| -----------------------------| ----------------|
| All visual / UI / frontend | [skills/frontend-design.md](skills/frontend-design.md) | always         |
| All server / API / data    | [skills/backend.md](skills/backend.md)                 | always         |
| Project state / onboarding | [code_of_conduct.md](code_of_conduct.md)               | always         |
| Audits (security/SEO/etc)  | [security.md](security.md)                             | when auditing  |
(Additional domains appended here as the collection grows)

### The prime rule

> **Design instructions come primarily from [skills/frontend-design.md](skills/frontend-design.md).**
> When a task involves any visual, typographic, color, layout, or interaction
> decision, open that file first and treat it as the spec. Do not substitute
> default styling, opinion, or "modern AI aesthetics" for what it says.

## 3. Non-negotiable rules

- **Design**: never generate "default AI aesthetics" (purple/indigo gradients,
  glassmorphism everywhere, pill badges, emoji-as-icons, `border-radius: 9999px`
  on everything, em-dash-heavy marketing copy) unless the design skill
  explicitly prescribes them. The skill decides the look. You enforce it.
- **Context**: gather context before editing. Read the relevant files, the
  industry, the audience. Design derives from the industry the project serves,
  not from trends.
- **Verification**: after any change, run the project's lint/typecheck/test as
  defined by the tooling (see `skills/` if defined) and report results.
- **Secrets**: never commit, log, or echo API keys, tokens, or credentials.
- **Scope**: do what is asked, no more. Do not refactor or restructure outside
  the task without asking.
- **State**: before every git commit, update [code_of_conduct.md](code_of_conduct.md)
  with what changed and any decisions made (short, newest on top). The next
  session resumes from that file, so it must be current.
- **Audits**: when an audit is requested, follow [security.md](security.md),
  then record the outcome in its state section so the next audit resumes where
  this one stopped.
- **Conventions**: match the existing code style, libraries, and patterns.
  Never assume a library is available. Check first.

## 4. How to extend this collection

When a new domain skill is added:

1. Create it under `skills/<domain>.md`.
2. Add a row to the **Domain authority map** above.
3. Keep every skill file self-contained and prescriptive (a decision engine,
   not a suggestion list), consistent with `frontend-design.md`.

## 5. Writing style for this collection

These files are the project's taste and rules, written once, reread every
session. They should be: precise, opinionated, and short. No fluff, no hedging,
no filler. Concrete over abstract, rules over vibes.