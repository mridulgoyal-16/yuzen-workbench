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
| Sheet / Expanded | Tap the floating **Search** pill | Search |
| Expanded | Drag the header down past 40px | Sheet |
| Search | Tap **✕**, or **done** on the keyboard | See [Search](#search) |
| Any workbench view | Tap **Tasks** in the nav | Tasks |

The bottom nav hides in search — the keyboard takes that space. The floating
Search pill shows only in `sheet` and `expanded`.

## Bottom sheet

Anchored to the bottom edge and **sized by its content**, capped at 668px:

| Favourites | Sheet height |
|---|---|
| 1–3 (one row) | 314px |
| 4–6 (two rows) | 452px |
| 7–9 (three rows) | 590px |

The sheet reserves 180px at the bottom so the floating Search pill and the
bottom nav both sit over its white surface.

### Grabber

A 44 × 4 rounded rectangle in `border/primary`, centred in a 32px band —
8px above it, 20px below. Tapping expands; a transparent hit area stretches the
tap target across the whole band. Dragging from the band moves the sheet:
rubber-banded upward (capped at 140px, damped to 55%), free downward.

## Expanded page

- **Favourites** section — up to nine tiles, with a pencil action in the header
- **Categories** — a sticky tab strip (Bike · Battery · IOT · Workflow), then
  that category's actions
- 24px between a section heading and its content; 36px between rows
- Scrolling to the bottom leaves room for the floating pill

Tabs scroll horizontally, and selecting one scrolls it into view.

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
- The pencil becomes a **Done** text button
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
