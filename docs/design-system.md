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
| Grabber band | 8px above the bar, 20px below |
| Search field vertical padding | 16px |

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
same kind of label. "Favourites" takes `content/primary`, "Recommended"
`content/secondary`.

Action labels are sentence-cased in JS, not CSS — see
[icons.md](icons.md#display-polish) for why.

## Layout constants

| Token | Value | Note |
|---|---|---|
| `--sheet-radius` | 24px | Sheet and expanded page top corners |
| `--ease` | `cubic-bezier(.32,.72,0,1)` | iOS-style sheet easing |
| `--morph` | 340ms | Pill → search field; JS reads this value rather than duplicating it |
| `--sheet-top` | measured | The sheet's live top edge; the expand animation starts here |
| `--scale` | measured | Fits the 390 × 844 frame to the window |

## Action tile

- 90px wide, three per row, evenly distributed
- 54 × 54 icon slot; the art sets the tile's height, not its width
- Label directly beneath, centred, clamped to two lines

## Notes for implementers

Two things worth knowing before porting this to Compose or SwiftUI:

**Everything is measured, not hardcoded.** The sheet's height follows its
content, the expand animation reads the sheet's live top edge, and the morph
duration is read from `--morph` rather than duplicated in JS. Keep that
discipline — the earlier hardcoded versions drifted and produced visible jank.

**The frame is scaled.** The prototype draws a 390 × 844 phone and scales it to
fit the window, so every measured rect is divided by that scale factor before
being used as geometry. That's a prototype concern only; it disappears on device.
