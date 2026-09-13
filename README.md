# Vayris website

The public-facing product site for Vayris — an open-source, local-first AI desktop agent.

Everything lives in **one file**: `index.html`. CSS, markup, and JS are inline. There is
no build step, no package manager, and nothing to install.

---

## Run it locally

```bash
python -m http.server 5173
# then open http://127.0.0.1:5173/index.html
```

Opening the file directly over `file://` works too, but a server is closer to production.

---

## Before you push

Two things in `index.html` are placeholders:

- **`github.com/DevanshLamba/vayris-ai`** — appears in the nav, hero, engine footer, all three
  "Get Vayris" paths, the close, and the footer. Swap for the real repo URL.
- **The Windows download href** — currently
  `https://github.com/DevanshLamba/vayris-ai/releases/download/v1.0.0/Vayris.Setup.1.0.0.exe`.
  That pattern always resolves to the newest release and starts the download immediately,
  but **only if the release asset is named exactly `Vayris.Setup.1.0.0.exe`**. Confirm the
  filename your build produces, then update both the href and the filename printed in the
  card above it.

---

## Copy accuracy

The copy deliberately avoids absolute claims the source can't back up. It does **not** say
inference is on-device, that nothing is ever sent anywhere, or that there is zero
telemetry. It says the agent loop, its memory, and your files stay on your machine, and
that the model runtime is your choice — local, or a provider key you supply.

If Vayris's actual behaviour differs, the claims live in exactly three places:

- the hero paragraph
- the "Where does your data actually go?" section, plus the note under the diagram
- the four module cards in "Under the hood"

The four modules (agentic engine, retrieval/memory, guardrails, voice input) and the
Electron shell should mirror what's actually in the repo. Rename or replace them if the
module layout changes. **Don't add a capability here before it exists there.**

---

## Structure

The page is written as six acts:

| Act | Section id | What it does                             |
|-----|------------|------------------------------------------|
| 01  | `top`      | Hero — the idea, both entry points       |
| 02  | `concept`  | The problem — cloud vs. local data path  |
| 03  | `credo`    | The philosophy — the statement moment    |
| 04  | `features` | The engine — module blueprint            |
| 05  | `get`      | The choice — three ranked paths          |
| 06  | `close`    | Open-source close + GitHub CTA           |

The left act rail, the nav active state, and the scroll progress bar all read from the
same `sections` array near the top of the script. **If you add or rename a section,
update that array and the `.rail` markup together**, or wayfinding silently drops it.

### The three entry points are deliberately unequal

Path 01 (run locally) is a full-width ember panel, Path 02 (beta .exe) is a half-width
card, Path 03 (browse source) is a quiet hairline block with a text link. The visual
weight *is* the recommendation — if you make them equal again, the section stops giving
advice.

---

## Design system

All colour and type decisions come from custom properties in `:root`:

| Token | Value | Used for |
|---|---|---|
| `--bg` / `--bg-alt` | `#0a0908` / `#100e0c` | page ground; alternating section tone |
| `--surface` | `#17140f` | cards, panels |
| `--ink` / `--ink-dim` / `--ink-faint` | `#f3eee6` / `#a79e92` / `#6e665c` | body copy, secondary, labels |
| `--ember` / `--ember-deep` / `--ember-glow` | `#ff8a3d` / `#b8481a` / `#ffb870` | the accent |
| `--steel` / `--steel-dim` | `#6fa3aa` / `#3e5c60` | technical details, cloud-side diagram |
| `--line` / `--line-soft` | 10% / 6% ink | borders, hairlines |
| `--font-display` | Fraunces | headlines (serif, italic for emphasis) |
| `--font-body` | Manrope | body |
| `--font-mono` | JetBrains Mono | eyebrows, labels, terminal, diagram |

**Orange is an accent, not a theme.** It marks the primary path, the active act, and one
italic phrase per headline. If a change makes a second element orange, something else
should probably stop being orange.

Fonts load from Google Fonts; each stack has a real system fallback, so the page is
readable before (or without) them.

Two things do **not** read from the tokens, so re-theming means touching them by hand:
the inline SVG diagram (SVG presentation attributes carry literal hex) and the Three.js
materials in the script. `--surface-hi` is defined but currently unused.

---

## Motion

Everything animates on entry, once, and only when it reaches the viewport.

- `.reveal` — fades and rises into place. Add `data-anim="left" | "right" | "fade"` to
  change the direction. Set `style="--d:.12s"` to stagger it inside a group.
- `.lines` — masked line-by-line headline reveal. Wrap each line as
  `<span><i>text</i></span>`; the script assigns the per-line stagger automatically.
- `.bp-line` — the blueprint connector draws itself across the module grid.
- `.diagram-wrap` — gets `.in-view`, which starts the travelling packets in the SVG.

A single `IntersectionObserver` (threshold `0.15`) adds `.in-view` and then unobserves,
so nothing re-animates on scroll-back. To animate something new, give it one of those
classes — don't add another observer.

**Under `prefers-reduced-motion: reduce`** all transitions and animations collapse to
~0ms, masked lines sit in place, and the reveal states never hide anything. Verified:
nothing is stranded invisible.

---

## The 3D hero

Three.js **r128**, pinned, from cdnjs — the only external script. The scene is a dark
ember-lit icosahedron core, an additive fresnel rim shell, a teal wireframe lattice
(open source, made literal), two thin orbital rings with a node running the inner one,
and a particle field.

Tuning knobs, all near the top of the scene setup:

| Variable | Now | Effect |
|---|---|---|
| `camBaseZ` | `9.4` | object size — raise to pull the camera back |
| `camDolly` | `2.2` | how far it recedes as you scroll past the hero |
| `particleCount` | `260` (`140` on small screens) | density of the field |
| `emissiveIntensity` | `0.30` | interior ember glow of the core |

Pointer input is an **eased offset** on top of the base rotation, never accumulated into
it — the object answers the cursor and settles back rather than drifting. The same
pointer drives camera parallax and the key light position.

Position and scale are CSS, not JS: `.hero-stage` is radially masked so the canvas edges
dissolve instead of showing a rectangle, and below 900px it drops behind and beneath the
copy at low opacity so type stays the priority.

**If WebGL is unavailable or Three.js fails to load, the script returns early** and the
page renders normally without the canvas. Nothing else depends on it.

---

## Responsive

| Breakpoint | What changes |
|---|---|
| `≥1440px` | the act rail appears |
| `≤940px` | Get Vayris collapses to one column; the quiet path gets a top border |
| `≤900px` | hero object moves beneath the copy at low opacity; modules go 2-up |
| `≤860px` | the concept section stacks, diagram offset removed |
| `≤720px` | nav links hide, GitHub button stays |
| `≤640px` | gutter drops to 22px, CTAs go full width, type scales down |
| `≤560px` | modules go 1-up |

Checked at 1440, 1280, 834, and 390 — no horizontal overflow at any of them. `html` and
`body` use `overflow-x: clip` so the bleeding hero object and the pre-reveal transforms
can't create a phantom scrollbar.

---

## Browser support

Modern evergreen browsers. Three features degrade rather than break:

- `:has()` — lights the blueprint node matching a hovered module. Without it, the node
  simply doesn't light.
- `overflow-x: clip` — falls back to `hidden` on the line above.
- `backdrop-filter` on the scrolled nav — falls back to the solid translucent background.

---

## Accessibility

- Focus rings on every button and link (`:focus-visible`, ember, 3px offset).
- The data-flow SVG has `role="img"` with `<title>` and `<desc>` describing both paths.
- Decorative layers (grain, vignette, progress bar, watermark) are `aria-hidden`.
- Full `prefers-reduced-motion` support (see Motion).
- Body copy is `--ink-dim` on near-black — keep new copy at that value or lighter.

---

## Performance

No framework, no bundler, one external script and one stylesheet. The render loop stops
entirely when the hero scrolls out of view or the tab is hidden, pixel ratio is capped at
2, and the particle count halves on small screens. Reveal observers unobserve after
firing. Keep it this way: if a change needs a library, it probably needs a rethink first.

---

## Host it free on GitHub Pages

1. In the repo on GitHub: Settings → Pages.
2. Under "Build and deployment", set Source to "Deploy from a branch".
3. Pick branch `main`, folder `/ (root)`, save.
4. It goes live at `https://<your-username>.github.io/<your-repo>/` within a minute or two.

To host it from a repo that already holds other things, drop `index.html` into that
repo's `/docs` folder and point Pages at `main` / `/docs`.
