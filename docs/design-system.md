# Design system

Everything is driven by CSS custom properties defined once at the top of
`index.html`, mapped from
[yulu-design-system](https://github.com/vaishnavijawdekar/yulu-design-system).

## Colour

Primitives:

| Token | Value |
|---|---|
| `--color-white` | `#FFFFFF` |
| `--color-grey-100` | `#F7F7F7` |
| `--color-grey-200` | `#E8E8E8` |
| `--color-grey-300` | `#DDDDDD` |
| `--color-grey-400` | `#B0B0B0` |
| `--color-grey-500` | `#919191` |
| `--color-grey-600` | `#717171` |
| `--color-grey-900` | `#222222` |
| `--color-green-500` | `#00654F` |
| `--color-orange-500` | `#E07912` |
| `--color-red-500` | `#C13515` |
| `--color-yulu-blue-500` | `#30CDFD` |

Semantic — always style against these, never the primitives:

| Group | Tokens |
|---|---|
| content | `primary` `secondary` `tertiary` `disabled` `inverse` `positive` |
| surface | `primary` `secondary` `disabled` `inverse` |
| border | `primary` `secondary` `selected` |
| canvas | `primary` `inverse` |

Notable uses: the bottom nav's unselected state is `content/tertiary`; the sheet
grabber is `border/primary`; dashed empty slots are `border/primary`, darkening
to `border/secondary` when pressed.

## Spacing

`4 · 8 · 16 · 24 · 36 · 48 · 64` as `--spacing-1` … `--spacing-16`.

Applied in this flow:

| Where | Value |
|---|---|
| Screen edge → content | 24px |
| Section heading → its content | 24px |
| Between action rows | 36px |
| Between action columns | 28px |
| Title → favourites (expanded page) | 36px |
| Favourites → category chips | 36px |
| Category chips → actions | 24px |
| Grabber band | 8px above the bar, 20px below |
| Search field vertical padding | 16px |

The 28px column gutter is the one number here that isn't a spacing token — see
[Action grid](#action-grid) for why.

## Type

Satoshi, loaded from Fontshare, with a system fallback stack.

Two cuts are loaded, 500 and 700. Satoshi has no semibold, so a role named
"600" maps to the **Bold** cut — the number is the token's name, not a CSS
weight. Don't write `font-weight: 600` against these faces: CSS font matching
rounds a request above 500 up to the next available weight, so it renders as
700 anyway, with nothing in the code saying why.

| Role | Size / line / weight | Used for |
|---|---|---|
| Headings / Small | 20 / 28 / 700 | defined, currently unused |
| Label / Medium (bold) | 16 / 20 / 700 | "My tasks" app-bar title |
| Label / Medium 600 | 16 / 20 / **700** | "Workbench" title |
| Label / Medium | 16 / 20 / 500 | "Favourites", "Recommended", Search pill |
| Label / Small | 14 / 16 / 500 | |
| Label / XSmall | 12 / 16 / 500 | action tile labels |
| Body / Medium | 16 / 24 / 500 | task rows |
| Body / Small | 14 / 20 / 500 | toast |

Section headings are label/medium, not Headings/Small: "Favourites" and
"Recommended" sit in the same slot on their respective screens and read as the
same kind of label, both in `content/primary`. The search label is one element
with two states — the "Recommended" prompt and the "N results" count — so they
share it.

Action labels are sentence-cased in JS, not CSS — see
[icons.md](icons.md#display-polish) for why.

## Layout constants

| Token | Value | Note |
|---|---|---|
| `--screen-radius` | 36px | The phone's own corners, both breakpoints |
| `--sheet-radius` | 36px | Sheet and expanded page top corners |
| `--ease` | `cubic-bezier(.32,.72,0,1)` | iOS-style sheet easing |
| `--morph` | 340ms | Pill → search field; JS reads this value rather than duplicating it |
| `--sheet-top` | measured | The sheet's live top edge; the expand animation starts here |
| `--scale` | measured | Fits the 390 × 844 frame to the window — desktop only; pinned to 1 on a phone |

## Viewport

Two modes, split at 800px — the same breakpoint the case styling uses.

**Below 800px the prototype is the screen.** The frame is `100dvh` tall and
full width, `.device` fills it with `inset: 0`, and there is no transform, no
outline and no corner radius. 390 × 844 is a desktop convenience — a stand-in
for a phone on a screen that isn't one. On a real handset it only gets in the
way: its aspect never quite matches the device, so fitting it leaves bands of
dead space, and the 2px stroke reads as a border drawn around the app rather
than the edge of a screen.

**At 800px and above** the fixed 390 × 844 box comes back, scaled to fit, with
the case ring and shadow.

Two things follow from this and are easy to get wrong:

- `100dvh`, not `100vh`. Mobile browsers keep reporting the *tallest* viewport
  for `vh` even while the URL bar is showing, which pushes the bottom nav below
  the fold until you scroll. `dvh` tracks the visible height as the bar moves.
  The `100vh` line before it is the fallback.
- **`deviceScale()` must return 1 in fluid mode.** It normally reports
  `rect.width / 390` to convert screen px to device px, but with the device now
  the full viewport that would return 1.10 on a 430px phone — silently scaling
  every drag threshold and measured rect by the handset's width ratio. There is
  no transform in that mode, so the answer is 1.
- **Nothing may assume a height of 844.** `padCatGrid()` did, and sized the
  short categories for a screen the handset doesn't have. It measures the
  device box instead — not the scroll container, which is mid-flight while the
  expanded page animates its `top` from the sheet's edge to 0.

The expanded page's status bar also swaps sides here: pinned in `.page__head`
on desktop, moved into the scroll on a phone so it scrolls away. See
[interactions.md](interactions.md#expanded-page).

## Action grid

`grid-template-columns: repeat(3, 1fr)` with a fixed `column-gap: 28px`,
spanning whatever the 24px screen margins leave.

**The gutter is the designed number; the column falls out of it.** Three equal
columns share the remaining width, so at 390px each is 95.33px
(`(342 − 2 × 28) / 3`) and on a wider handset they simply grow. The gutter
stays 28px at every width.

Don't pin the columns to 95.33px and reach for `justify-content: space-between`
instead. That produces identical pixels at exactly 390px and nowhere else: the
prototype runs full-bleed on a phone now, so on a 430px device the spare width
has nowhere to go but the gutters, and they open up to 48px.

One grid everywhere: the sheet and the expanded page share identical metrics,
so the expand animation doesn't re-lay-out its contents mid-motion.

### Action tile

- Equal thirds — 95.33px at 390px wide — with 28px gutters, three per row
- 102px tall: a 54 × 54 icon slot, a 16px gap, then a 32px label box
- A 3D icon fills the 54px slot outright — no circle behind it. Actions with no
  art yet keep a 48px placeholder circle, centred in the same slot, so rows stay
  level when a grid mixes the two
- Label directly beneath, centred, clamped to two lines. The box is a fixed
  32px whether the label wraps or not, so row heights never vary
- Empty slots stay a 90px square and are centred in their column with
  `justify-self` — a fixed-width grid item doesn't stretch, so without it the
  slot sits at the column's start edge, out of line with the icons above

## Category chip

Chips, not underlined tabs.

- 48px tall, 16px horizontal padding, fully rounded
- 12px between chips, 24px from the screen edge
- 24px icon, 8px gap, label/medium
- Selected: `content/primary` fill, `content/inverse` label
- Unselected: `surface/inverse` fill inside a `border/primary` hairline

The strip scrolls horizontally and carries no trailing fade or underline — it's
opaque, so pinned content scrolls cleanly beneath it.

## Notes for implementers

Two things worth knowing before porting this to Compose or SwiftUI:

**Everything is measured, not hardcoded.** The sheet's height follows its
content, the expand animation reads the sheet's live top edge, and the morph
duration is read from `--morph` rather than duplicated in JS. Keep that
discipline — the earlier hardcoded versions drifted and produced visible jank.

**The frame is scaled.** The prototype draws a 390 × 844 phone and scales it to
fit the window, so every measured rect is divided by that scale factor before
being used as geometry. That's a prototype concern only; it disappears on device.

**Never write `font: <weight> <size>/<line-height> inherit`.** The `font`
shorthand ends in a font-family and `inherit` isn't one, so the declaration is
invalid and the browser throws all of it away — size included. On a `<button>`
the fallback is 13.33px, which is small enough to look like a design choice
rather than a bug. It sat in the coded keyboard undetected until someone
measured: the stylesheet said 22px, the keys rendered at 13.33px. Use longhands
(`font-weight` / `font-size` / `line-height` / `font-family: inherit`) anywhere
the family needs to be inherited.
