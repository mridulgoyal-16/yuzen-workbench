# Interactions

Every screen, gesture, and transition in the prototype.

## View model

One state variable, four values: `tasks` · `sheet` · `expanded` · `search`.

The sheet stays mounted behind everything workbench-y. The expanded page is not
a separate screen that fades in — it starts at the sheet's own top edge with the
sheet's 24px corner radius and stretches to full screen, so the white surface
and its corners stay continuous the whole way.

| From | Action | To |
|---|---|---|
| Tasks | Tap **Workbench** in the nav | Sheet |
| Sheet | Tap the grabber, or drag up past 48px | Expanded |
| Sheet | Drag down past 28% of sheet height | Tasks |
| Sheet | Tap outside (scrim) | Tasks |
| Sheet / Expanded | Tap the **Search** pill | Search |
| Expanded | Drag the header down past 40px | Sheet |
| Search | Tap **✕**, or **done** on the keyboard | See [Search](#search) |
| Any workbench view | Tap **Tasks** in the nav | Tasks |

The bottom nav hides in search — the keyboard takes that space. The Search pill
shows only in `sheet` and `expanded`.

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

1. Grabber
2. **Workbench** title
3. **Favourites** heading
4. The favourites themselves
5. **Search** pill

The title and heading are the same components the expanded page uses, at the
same sizes, so the sheet reads as the top of that page rather than a different
screen. The heading carries no pencil — editing belongs to the expanded page.

**Sized by its content**, capped at 620px. Fixed chrome is 242px — 8px above
the grabber bar, the 4px bar, 16px from the bar to the title, the 20px title,
26px under it, the 48px heading, 24 above the icons, 24 below them, the 48px
pill, 24 to the sheet's bottom edge — plus 102px per row of favourites and 36px
between rows:

| Favourites | Sheet height |
|---|---|
| 1–3 (one row) | 344px |
| 4–6 (two rows) | 482px |
| 7–9 (three rows) | 620px |

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

### Grabber

A 44 × 4 rounded rectangle in `border/primary`, 8px below the sheet's top edge
and 16px above the Workbench title. Tapping expands. The bar carries a transparent
**72 × 44** hit target, reaching from the sheet's top edge down past the band
into the title's empty upper margin. Dragging from the band moves the sheet:
rubber-banded upward (capped at 140px, damped to 55%), free downward.

## Expanded page

- **Favourites** section — up to nine tiles, with a pencil action in the header
- Page title **Workbench** — label/medium in `content/secondary`, carried
  through to the search screen so the header doesn't shift
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

### Results

Results carry no category context, so — exactly like favourites — they use each
action's full name and its `_common` icon. Two actions named "Status" would
otherwise be indistinguishable.

### Closing

The large **✕** returns you *at least* as far as you came from:

| Opened from | Typed something? | Lands on |
|---|---|---|
| Expanded page | either | Expanded page |
| Sheet | yes | Expanded page |
| Sheet | no | Sheet |

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
| Browse everything in a category | Workbench → grabber → tab |
| Save an action for later | Expanded → pencil → **+** on the action |
| Clear out a stale favourite | Expanded → pencil → **−** on the favourite |
| Fill an empty slot | Sheet → tap the dashed **+** → lands in edit mode |
| Back out of search without losing your place | **✕** — see the table above |

## Not built

Action screens. Tapping any action shows `<name> — screen not built yet`.
Favourites are in-memory and reset on reload.
