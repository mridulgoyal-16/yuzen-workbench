# Icon system

## Folders

```
3d icons/
  bike/       battery/      IOT/      workflow/
```

One folder per category. Every file is a 54 × 54 PNG. These are the **action**
icons — the 3D art on the tiles.

`assets/` holds the flat 2D art: bottom-nav mask icons and the cancel glyphs.

`Icon style/yulu-icon-style_final.json` holds the icon style definition.

## Category chip glyphs

The four category chips do **not** use files. Their icons are inline 24 × 24
SVGs held in `CAT_GLYPH` in `index.html`, drawn with `fill: currentColor` so a
chip's icon inverts with its label when the chip is selected — a PNG can't do
that without a second asset or a mask.

That replaced an earlier arrangement of 32 × 32 PNG tab icons
(`assets/bike.png`, `battery.png`, `IOT.png`) plus `assets/workflow.svg`, which
was a monochrome placeholder that never matched the other three. **All four of
those files are now unreferenced** and can be deleted; they're left in place
only because nothing depends on their absence.

To change a chip's icon, replace the SVG path in `CAT_GLYPH`. Keep the
`class="tab__icon"`, the 24 × 24 viewBox, and `fill="currentColor"`.

## `_tab` and `_common`

Each action can have two variants:

| Suffix | Used where |
|---|---|
| `_tab` | In a category's action grid |
| `_common` | Favourites, search results |

(The `_tab` name predates the switch from tabs to chips. It's the filename
suffix on disk, so it stays as it is.)

If only one file exists it serves both places. Most actions ship `_common` only;
`_tab` art exists where the two contexts want visibly different treatment.

A file with no suffix at all (e.g. `IOT device info.png`) also serves both.

## id vs label

Every action has two strings, and they are deliberately different:

- **id** — the icon filename stem. Globally unique. This is the lookup key.
- **label** — what the tile displays.

The rule:

> Inside a category the category is implied, so the label drops the qualifier.
> In favourites and in search there is no category context, so the action
> reverts to its full id — and the icon switches from `_tab` to `_common` on
> the same flag.

| id | In the Bike category | In favourites / search |
|---|---|---|
| `bike status` | Status | Bike status |
| `report bike` | Report | Report bike |
| `bike device swapping` | Device swapping | Bike device swapping |
| `bike device info` | Device information | Bike device information |
| `battery status` | Status | Battery status |
| `report battery` | Report | Report battery |
| `remove battery mapping` | Remove mapping | Remove battery mapping |

**Why ids must stay unique:** Bike and Battery both have a "Status" and a
"Report". Before ids and labels were separated they collided and shared one
icon — the Bike tile rendered the battery art.

## Display polish

The stored strings have to match the filenames byte for byte — they're the
lookup keys — so every cosmetic fix happens in `display()`, at the last moment
before a name is drawn:

| Rule | Example |
|---|---|
| Sentence case | `bike status` → "Bike status" |
| `IOT` → `IoT` | `IOT device info` → "IoT …" |
| `E- bike` → `E-bike` | `E- bike battery swapping` → "E-bike battery swapping" |
| `device info` → `device information` | `IOT device info` → "IoT device information" |
| `Miracle BLE reset` → `BLE reset` | the icon file keeps the longer name |

Search matches against both forms, so typing either `e- bike` or `e-bike` finds
the action.

Sentence case used to be a CSS rule (`.tile__label::first-letter`). That
pseudo-element doesn't apply to the `-webkit-box` the label needs for its
two-line clamp: Chrome blockifies the box and honoured it anyway, iOS Safari
didn't, and every label rendered lowercase on device. Keep it in JS.

`WRAP_AFTER` forces a line break in a label that would otherwise sit awkwardly
— either squeaking onto one line beside a row-mate that wrapped, or breaking in
the wrong place. Keyed on the *displayed* name; the value is what ends line one:

| Displayed | Breaks as |
|---|---|
| MRE verification | MRE / verification |
| Correct mapping | Correct / mapping |
| Bike deployment | Bike / deployment |
| YCS swapping | YCS / swapping |
| IoT device information | IoT device / information |

`MRE verification` is the original case: at 83px against an 86px label it just
fit on one line while its row-mate `Bike assessment` wrapped, and the row read
lopsided.

## Adding an action

1. Export the art into the right category folder. Name the file after the
   action, adding `_common` (and `_tab` if the tab needs different art).
2. Add the id to that category's `actions` array in `index.html`.
3. Add an `ART` entry pointing at the file(s).
4. If the category grid should show a shorter label, add a `LABEL[id]` entry.
   Skip it and the full id is used in both places.

Two rules to keep:

- **Never let two actions share an id.** It is the icon key.
- **Name the id after the file**, not after the display text.

## Known quirks

Filenames are taken verbatim from the export, typos included, so the code maps
around them rather than renaming:

| Action | File |
|---|---|
| Deploy battery on bike | `Deploy batterty_common.png` |
| Battery handshake (common) | `battery handhake_common.png` |
| MRE verification | `mreverification_common.png` |

`Bike device info_tab.png` is cased differently from every other file in that
folder — capital B, where its `_common` sibling is lowercase. Pages serves
case-sensitively, so the `ART` entry has to match exactly.
