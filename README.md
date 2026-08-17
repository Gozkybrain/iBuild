# iBuild

Guidance files that teach an AI your standards before it touches your code.

Most people start a project with an AI by telling it what to do on the spot.
Every session starts over. iBuild is the fix: a small set of plain-text files
at the root of your project that describe how you want things done, once, so
every AI session and every AI tool follows the same rules from the first
line.

---

## What a README does

The README is the front door of a project. It is the first file a person
opens and often the first file an AI reads too. Its job is not to repeat
documentation. Its job is to answer three questions fast:

1. **What is this project?**
2. **Why does it exist?**
3. **Where do I go next?**

If those three questions are not answered in the first screen of text, the
README has failed. Everything else on a project page is detail for later; the
README is the summary you need right now.

---

## What is in the box

Each file has one job. Together they form the full briefing an AI needs.

### [AGENTS.md](AGENTS.md) (the index)

The rulebook. It tells the AI how to behave everywhere: which rules always
bind, which file has authority over which domain, and what must happen
before every commit. It is the first file an AI reads, and it points to
everything else.

### [`skills/`](skills/) (the specialist guides)

One file per craft, written as a decision engine, not a suggestion list.

- [frontend-design.md](skills/frontend-design.md) says what good design is
  for this project and what to never ship (no generic AI looks, no purple
  gradients, no emoji icons).
- [backend.md](skills/backend.md) says how the server side should be built
  and how to stay inside free-tier quotas, so the project stays cheap while
  it grows.

The AI checks the relevant skill before doing any work in that area.

### [code_of_conduct.md](code_of_conduct.md) (the live record)

The project's memory. Before every commit, the state of the project is
written here: what changed, what was decided, where things stand. A brand new
AI can read this one file and pick up where the last session stopped, without
re-reading the whole codebase.

### [security.md](security.md) (the audit book)

The checklist the AI runs when you ask for an audit: secrets, known
vulnerabilities, access rules, SEO, performance. After each audit it records
where it stopped, so the next audit resumes where the previous one left off.

---

## Why the README matters, even with docs present

Documentation describes how the project works. The README has a different
job, and a project with good docs but a bad README still fails:

- **It sets the frame.** Docs assume you already know what you are looking at.
  The README tells you what you are looking at and why it exists. Without it,
  even complete documentation is a map with no legend.
- **It is the shared entry point for humans and AI.** When both read the same
  one-page framing first, a human and an AI tool start from the same
  understanding. That is the cheapest possible context sync.
- **It is a guarantee a project is alive.** Projects without a README feel
  abandoned or internal. A clear README is the marker of a project meant to be
  used by others.
- **It forces the thing to be explainable.** Writing the README in plain
  words exposes when a project has no clear purpose. If it cannot be
  explained simply, it cannot be built well.

---

## How it works together

```
You (one page of intent)
        \                                  AI tool reads, in order:
         v                                 1. AGENTS.md (the rules)
+---------------------+                    2. code_of_conduct.md (the state)
|   The README sets   |                    3. skills/ (its craft)
|   your expectations |                    4. security.md (before audits)
+---------------------+                    5. then it starts working
         |
         v
   Every file above
   + your codebase
```

That order matters. The AI reads the rules first, then the state file, so it
knows both how to behave and where things stand before touching anything. It
only opens the skill for the domain it is working in, and only opens
security.md when an audit is asked for. Nothing runs, nothing installs; the
files are read by the AI, kept current by the AI, and understood by your
team.

The one rule you must follow: never let a commit happen without the state
file being updated first.

## Running an audit

Whenever you want the project checked, ask the AI to run an audit. The full
checklist lives in [security.md](security.md); it covers secrets, known
vulnerabilities, access rules (security rules / RLS), SEO, performance, and
quota. The AI follows that file, reports every finding with severity and fix,
and records where it stopped, so the next audit resumes where this one ended.

The audit prompt:

> Run an audit on this project following [security.md](security.md).

A clean audit is not assumed; the AI says what was checked, what failed, and
what was not covered. The result also lands in
[code_of_conduct.md](code_of_conduct.md), so the project graph always shows
the last audit state.

## Building the project

The app's structure, stack, navigation, auth, and API contract are defined in
[documentation.md](documentation.md). To build it, ask the AI to scaffold
using that file:

> Set up a fullstack Next.js app for me using the folder structure in
> [documentation.md](documentation.md).

The AI creates the whole tree with dummy data, installs only the dependencies
listed there, applies the design rules, and then stops for you to customize.
It does not invent files or features beyond the spec.

## Using this on your own project

1. Copy the files into the root of your project.
2. Edit the [README.md](README.md) to describe your project, not this one.
3. Point your AI tool at the repository and start building.
4. Let it keep [code_of_conduct.md](code_of_conduct.md) and
   [security.md](security.md) current as it works.

You do not need to re-teach your AI what you like. It is already written down
for you.