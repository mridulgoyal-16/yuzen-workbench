# Interactions

Every screen, gesture, and transition in the prototype.

## View model

One state variable, four values: `tasks` · `sheet` · `expanded` · `search`.

The sheet stays mounted behind everything workbench-y. The expanded page is not
a separate screen that fades in — it starts at the sheet's own top edge with the
sheet's 36px corner radius and stretches to full screen, so the white surface
and its corners stay continuous the whole way.

| From | Action | To |
|---|---|---|
| Tasks | Tap **Workbench** in the nav | Sheet |
| Sheet | Tap the arrow, or drag up past 48px | Expanded |
| Sheet | Drag down past 28% of sheet height | Tasks |
| Sheet | Tap outside (scrim) | Tasks |
| Sheet / Expanded | Tap the **Search** pill | Search |
| Expanded | Drag the header down past 40px | Sheet |
| Search | Tap **✕**, or **done** on the keyboard | See [Search](#search) |
| Any workbench view | Tap **Tasks** in the nav | Tasks |

The bottom nav is **never unmounted** — the keyboard simply covers it. The
search page sits above the nav in the stack (z 60 vs 50) and paints an opaque
background over it, so the nav is out of sight and unclickable while search is
open, and is uncovered again as the keyboard leaves. Hiding the nav and
restoring it at the end of the close left an empty band at the bottom of the
screen for the whole time the keyboard was sliding away.

The Search pill shows only in `sheet` and `expanded`.

## Collapsed screen

Four bands, bottom-anchored, nothing below the last one:

```
Tasks content          ← stationary, dimmed by the scrim
Workbench sheet        ← overlays the tasks, never pushes them up
Bottom nav             ← fixed at the physical bottom, 100px
Home indicator         ← inside the nav's band, 8px off the edge
```

### Bottom sheet

Its bottom edge sits on the **nav's top edge**, not the screen's. Running the
white surface underneath the nav made the collapsed screen read as two screens
stacked — favourites on one, search and nav on another.

Everything Workbench-y lives inside the sheet, in this order:

1. Expand arrow
2. **Favourites** heading
3. The favourites themselves
4. **Search** pill

The heading is the same component the expanded page uses, at the same size, so
the sheet reads as the top of that page rather than a different screen. It
carries no pencil — editing belongs to the expanded page.

**Sized by its content**, capped at 618px. Fixed chrome is 240px — 36px above
the arrow, the 14px arrow, 36px from the arrow down to "Favourites", the rest
of the 48px heading, 24 above the icons, 24 below them, the 48px pill, 24 to
the sheet's bottom edge — plus 102px per row of favourites and 36px between
rows:

| Favourites | Sheet height |
|---|---|
| 1–3 (one row) | 342px |
| 4–6 (two rows) | 480px |
| 7–9 (three rows) | 618px |

Nine favourites is the most possible, so the cap never actually bites — it's a
guard. Every child is `flex: none`: a column flex container that hits its cap
otherwise squashes whichever children can give, and the heading silently
collapsed from 48px to 32px at nine favourites before that was pinned.

### Search pill

One button with two homes. Inside the collapsed sheet it's the last row of the
sheet's own content — static, no gradient scrim. On the expanded page it floats
over the scroll with its scrim, 164px off the bottom. The node is *moved*
between the two rather than duplicated, so the pill → field morph keeps
measuring a single live element.

Floating, it carries `0 5px 18px rgba(0,0,0,.15)` plus a tight contact shadow.
That reach — 5 + 18 = 23px below the pill — is the most it can have: inside the
collapsed sheet only 24px of padding sits under it before `overflow: hidden`
clips the tail. To make it heavier, raise the opacity, not the blur.

**Landed at the end of the scroll.** Once the page can't scroll further there
is nothing behind the pill: the scrim has nothing to fade out and the lift has
nothing to lift off. Both come away over 200ms and the pill reads as sitting on
the bottom of the page rather than hovering over it. The scrim is on a
pseudo-element purely so it can fade — gradients don't interpolate to `none`,
opacity does. In the sheet the pill is a plain row and never lands.

This was picked over keeping the pill always-floating. To undo it, delete the
two `.searchfab.is-landed` rules; the JS that toggles the class is then inert.

### Expand arrow

A 24 × 14 chevron in `content/disabled` (#B0B0B0), 36px below the sheet's top
edge and 36px above "Favourites". Tapping expands. It carries a transparent
**52 × 44** hit target — the glyph alone is far too small to aim at, and the
padding around it is dead space otherwise. Dragging from the band moves the
sheet: rubber-banded upward (capped at 140px, damped to 55%), free downward.

## Expanded page

- **Favourites** section — up to nine tiles, with a pencil action in the header
- Page title **Workbench** — label/medium600 (Satoshi Bold 16/20) in
  `content/primary`, carried through to the search screen so the header
  doesn't shift
- **Categories** — a sticky tab strip (Bike · Battery · IoT · Workflow), then
  that category's actions
- 24px between a section heading and its content; 36px between rows
- Scrolling to the bottom leaves room for the floating pill

Every tab shows its 32 × 32 icon, selected or not; selection reads from the
underline and the darker label. Four tabs with icons overflow the 390px screen,
so the strip scrolls horizontally and selecting a tab scrolls it into view.

### Keeping the strip pinned

`position: sticky` only holds while the page stays tall enough to remain
scrolled that far. IoT has one action, so switching to it from the bottom of
Bike shrank the page below its own scroll offset — the browser clamps
`scrollTop` and the strip slides back down the screen mid-read.

Two things hold it still:

- A short category's grid grows a `min-height` sized so the scroll ends
  **exactly** where the strip pins — no less, or the strip slides back down; no
  more, or those one or two actions scroll up underneath it and get cut off. At
  the end of the scroll the first row sits 24px below the strip, same as it
  would unscrolled.
- `scrollTop` is held across the whole swap. Rebuilding the strip and the grid
  each momentarily shrink the page and the browser clamps on the spot;
  restoring the height afterwards doesn't bring the scroll back.

Two measurement traps, both of which produced a padding that drifted on every
switch until it was wrong:

- **`offsetTop` on a `position: sticky` element reports where it's painted**,
  not its place in the layout. Read while the strip was pinned, it returned the
  scroll offset plus the header, and the padding compounded — 590px, then
  706px. Everything is derived from `#catGrid` instead, which is statically
  positioned.
- **`scrollHeight` bottoms out at `clientHeight`.** On a short category it
  reports the port height, so a tail measured as `scrollHeight − content` came
  out 18px too big. The tail is read off the container's `padding-bottom`
  instead; the grid is its last child.

## Search

### Opening

The floating pill and the search field are the same surface. On open the field
is measured, planted on the pill's exact rect, then released to its own
geometry — so it grows and travels upward in a single 340ms move while the
keyboard rides up underneath. Closing runs the same move backwards.

### Typing

- Coded iOS keyboard: letters, numbers, symbols, shift, backspace, done
- Results filter live on every keystroke
- Matching is on an action's **full name**, so typing `bike` finds the Bike
  actions even though their tab labels drop the word
- Up to 12 results; an empty query shows six recommended actions
- A **✕** inside the field clears the query without leaving search
- No matches shows `No actions match "…"`

The empty field reads **Search**, in `content/disabled` (#B0B0B0). Its magnifier
is `content/primary`, matching the floating pill's — the pill morphs into this
field, so an icon that changed shade mid-flight reads as two different icons.

**Recommended** occupies the same box as **Favourites** on the expanded page —
48px tall, same padding, same label/medium type, and the grid below it carries
the same 24px `gap-head`. Both the label and the first tile land on identical
pixels, so nothing jumps when search takes over the screen.

### Results

Results carry no category context, so — exactly like favourites — they use each
action's full name and its `_common` icon. Two actions named "Status" would
otherwise be indistinguishable.

### Closing

The large **✕** always lands on the **expanded page**, never back in the sheet,
whichever screen search was opened from. Coming out of a search you're looking
for something, and the sheet holds only nine favourites — landing there hides
the very catalogue you were searching.

Two consequences the code handles explicitly:

- The field morphs onto where the pill **will be**, not where it was when
  search opened. Opening from the sheet captures the in-sheet pill, which sits
  8px above the floating one. The pill is parked in its destination and
  measured with `visibility: hidden` — that still lays out, unlike `hidden`,
  which measures zero.
- When search was opened from the sheet, the expanded page grows **during** the
  morph rather than after it. Left to the view swap you'd sit through 340ms of
  morph and then a further 420ms of the page climbing. Overlapped it's one
  move, ~390ms; the closing search page is transparent, so the page shows
  through as it grows.

## Edit mode

Lives on the expanded page only; leaving that page exits it.

- Enter via the pencil in the Favourites header, or by tapping a dashed **+**
  slot (from the sheet, that jumps to the expanded page and opens edit mode)
- The pencil becomes a **tick**. Both are 24px icons in the same 48px slot, so
  the header doesn't reflow on the swap the way a text button made it
- All tiles wiggle, iOS-style
- Each favourite gets a **−** badge; tapping removes it
- Each catalogue action gets a **+** badge; tapping adds it to favourites

Guard rails, each with a toast:

| Situation | Toast |
|---|---|
| Action already a favourite | `<name> is already a favourite` |
| Nine favourites reached | `Favourites are full — remove one first` |
| Added | `<name> added to favourites` |
| Removed | `<name> removed` |

### Empty slots

A partial row is padded out to three with dashed slots — 8px dashes, 8px gaps,
drawn in SVG so the dash length is controllable. Counts of 1, 2, 4, 5, 7 and 8
leave a gap; 3, 6 and 9 are complete rows and get none. Zero favourites shows a
full empty row.

## Use cases

| Goal | Path |
|---|---|
| Run a frequent action | Workbench → tap it in the sheet |
| Find an action you don't have saved | Workbench → Search → type → tap the result |
| Browse everything in a category | Workbench → arrow → tab |
| Save an action for later | Expanded → pencil → **+** on the action |
| Clear out a stale favourite | Expanded → pencil → **−** on the favourite |
| Fill an empty slot | Sheet → tap the dashed **+** → lands in edit mode |
| Back out of search without losing your place | **✕** — see the table above |

## Not built

Action screens. Tapping any action shows `<name> — screen not built yet`.
Favourites are in-memory and reset on reload.
