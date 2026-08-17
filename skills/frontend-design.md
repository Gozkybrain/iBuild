# Frontend Design Skill

The authoritative spec for every visual decision in this project. AGENTS.md points
here. If you are working on anything the user can see, you are working from this
file. Read it fully before touching a stylesheet.

## 1. What we are trying to do

Build interfaces that look designed *for an industry and audience*, not
"modern", not "AI-generated", not "futuristic as a default". The result should
read as: deliberate, human, and specific to the product.

A design is successful when it could plausibly be the work of a senior
contractor who was briefed on that business. It fails when it looks like every
other AI landing page.

## 2. The markers of "AI design" we never ship

These are telltale signs, not crimes in themselves, but if more than one shows
up, the design is generic and wrong. Actively avoid:

- **Purple / indigo gradients** as a hero treatment or default accent
  (Laravel-purple, "vibe" gradients). If you feel the need for a gradient,
  prefer subtle, same-hue, dark-to-black depth instead.
- **Emoji as icons**. Icons are drawn SVG. Emoji belong in chat, not UI.
- **`border-radius: 9999px` on everything** (pill-shaped buttons, chips,
  inputs). Real products use a rounded-rectangle language, with one radius
  scale, not a universal pill.
- **Glassmorphism** (`backdrop-filter: blur` frosted cards) used as a default.
  Reserved for very specific overlay moments (modals over content).
- **Em-dash-heavy marketing copy** ("Capture moments effortlessly."). Write
  like a human, not a manifesto.
- **The generic hero**: 60px bold sans headline, left-aligned, paragraph of
  filler, two buttons (one gradient, one outlined), 900px mockup below. If the
  layout could serve any startup, it serves none.
- **Bootstrapped "modern" font stack only** (Inter + system-ui everywhere with
  no hierarchy plan).
- **Animated on everything** just to feel alive. Motion must have a reason
  (orientation, feedback, hierarchy), not flourish.

## 3. Design starts from the industry

Before writing any CSS, answer: *What does the customer in this industry trust?*

- **Fintech / banking**: trust, calm, precision. Restrained color (deep navy,
  graphite, one strong accent). Tight typography, tabular numerals. No playful
  animation. Forms feel solid, not fun.
- **Healthcare**: clean, human, no trickery. Whitespace is the luxury. Blues
  and teals over saturated hues. Generous without being childish.
- **Legal / accounting**: quiet authority. Serif display or classic sans.
  Monochrome first, one accent. Very little decoration.
- **SaaS / dev tools**: respect the technical user. Dense but organized.
  Dark or light both fine, but consistent. Monospace for code/uses, real
  data-density, focus on keyboard/task workflows.
- **E-commerce / retail**: product-forward. Whitespace around product imagery,
  colors from the inventory, trust badges handled as information not as gaudy.
- **Fashion / creative**: editorial. Big type, strong grids, generous image
  scale. Inverted palettes are fine. Texture (paper, grain) acceptable sparingly.
- **Gaming**: energy, but not kitsch. Deep darks, electric accent, kinetic.
  Avoid generic "fantasy purple".

If the industry is not listed: study how the *market leaders* in that vertical
present themselves and derive the rules; do not invent a theme.

## 4. Color system

- Start **neutral-first**: a real off-white light / near-black dark, plus the
  full gray ramp. Neutrals carry the design.
- Pick **one accent** derived from the brand or industry trust signals. Use it
  sparsely: primary CTA, active states, emphasis.
- Avoid the neon-gradients-animated-purple family unless the brand IS that
  (rare).
- Never hardcode colors inline; define a token system
  (`--color-bg`, `--color-fg`, `--color-mute`, `--color-line`, `--color-accent`,
  plus semantic `--color-danger`, `--color-success`).
- Ensure contrast passes WCAG (4.5:1 for body). Accessible is part of the
  design, not an afterthought.

## 5. Typography

- Choose a system with intent: one display/headline face and one body face,
  chosen by industry. Serif for authority/editorial, grotesque sans for
  utilitarian work, geometric for playful.
- Define type scale (usually 5–7 steps) with explicit sizes, line-heights,
  and letter-spacing. Headlines tight (`-0.02em` to `-0.04em`), body normal.
- Tabular/figures for numbers when financial.
- In dark mode bump line-height; avoid pure black.
- Never default to Inter + "futuristic" casing gimmicks (all-caps tracking zero)
  without a reason.

## 6. Layout & spacing

- Use an 8px (or 4px) spacing scale; multiples only. Consistent rhythm beats
  creativity.
- Max content width ~1100–1400px with generous margins; center or align to
  azimuth consistently.
- Use asymmetry and whitespace as tools. A centered everything layout reads
  generic.
- Real hierarchy: one primary element per section, not everything big.
- Use a consistent, small set of components (cards, buttons, inputs) rather
  than bespoke one-offs per feature.

## 7. Components & motion

- **Buttons**: solid primary, neutral secondary, minimal ghost. Consistent
  radius (not universally 9999px), consistent height, clear hover/focus
  states.
- **Cards**: 1px border + soft shadow (never heavy drop shadows + lots of blur).
  Border and background should blend rather than pop.
- **Icons**: inline SVG, consistent stroke weight, exported as system icons.
- **Motion**: 150–300ms ease cubic bezier for hovers/focus, 200–500ms for
  layout. Respect `prefers-reduced-motion`. Every animation must answer "what
  does this tell the user?"
- **Elevation**: shadows for elevation over content; never on background panels.

## 8. Content voice

- Copy is specific, confident, and reads like an actual person wrote it.
- No "Elevate your workflow". Write what the product *does*.
- No em-dash-as-loudspeaker. Use sentence short rhythm. Commas and periods.
- Real examples, concrete numbers ("3 ms response") over adjectives.

## 9. Verification checklist

Before considering frontend work done:

- [ ] No AI-aesthetic markers from section 2 leaking in.
- [ ] Colors from tokens; contrast passes for body text.
- [ ] Type from the defined scale; hierarchy is explicit, not accidental.
- [ ] Spacing on the scale; layout is asymmetric where it matters, aligned where
      it must.
- [ ] Copy reads human; concrete over abstract, no manifesto.
- [ ] Component reuse: no three different button styles for the same action.
- [ ] Runs clean through the project's lint/typecheck/test.

## 10. Reference anchors

When in doubt, ask: what would *this* industry's best example look like? Look
for the standard-bearers (e.g., Stripe for developer tooling trust, Linear for
task density, Apple for restraint in consumer). Study them, then design the
industry, not a copy of the reference.