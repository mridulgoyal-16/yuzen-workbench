# Yuzen Workbench

Clickable prototype of the **Workbench** tab for the Yulu field-ops app. One HTML
file, no build step, no dependencies — open it and the whole flow works: bottom
sheet, expand, search with a real keyboard, and favourites editing.

**▶ [Open the prototype](https://mridulgoyal-16.github.io/yuzen-workbench/)**

Runs in any modern browser, desktop or mobile. The phone frame scales to fit the
window, so it's fully usable on a laptop.

---

## What's in here

| Path | What it is |
|---|---|
| `index.html` | The entire prototype — markup, tokens, styles, and logic |
| `assets/` | Flat PNGs: tab icons, nav icons, cancel glyphs |
| `3d icons/` | Action art, one folder per category (`bike`, `battery`, `IOT`, `workflow`) |
| `Icon style/yulu-icon-style_final.json` | Icon style definition |
| `docs/` | Interaction spec, icon system, design-system mapping |

## Running it locally

Open `index.html` in a browser. That's it.

Some browsers block `file://` requests for the mask images used by the bottom
nav. If the nav icons come out blank, serve the folder instead:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000`.

## Sources of truth

- **Design system** — [yulu-design-system](https://github.com/vaishnavijawdekar/yulu-design-system) (tokens, components, Satoshi)
- **Figma** — file `35bYJ0YXYUW1NNu8xmERM8`, *Yuzen profiles / features*
  - `1073-14289` bottom sheet · `1073-14765` search · `1073-15248` expanded · `1073-16186` expanded + scrolled · `1125-14694` grabber

---

## The flow in one minute

Four views, one continuous white surface:

```
  Tasks ──tap Workbench──▶ Sheet ──grabber / drag up──▶ Expanded
                            │                             │
                            └──────── Search ◀────────────┘
```

**Sheet** — favourites only, sized to its content. Five favourites make a
shorter sheet than nine; there's never dead space under the last row.

**Expanded** — favourites plus the full catalogue under category tabs. The sheet
doesn't cross-fade to a new page; it grows into one, so the surface and its
corner radius stay continuous.

**Search** — the floating Search pill *becomes* the search field. Same surface,
one move, keyboard rides up underneath.

## Features

### Search

- Floating pill, always within reach above the bottom nav
- Tapping it morphs the pill into the search field (iOS Spotlight-style)
- A coded iOS keyboard — letters, numbers, symbols, shift, backspace
- Results filter live as you type
- Results always use an action's **full name** and its `_common` icon, so two
  actions called "Status" stay distinguishable
- Empty query shows six recommended actions
- Closing returns you at least as far as you came from — see
  [docs/interactions.md](docs/interactions.md#search)

### Favourites and editing

- Up to nine favourites, three per row
- A partial row is padded with dashed **+** slots; tapping one opens edit mode
- Edit mode (pencil → Done) wiggles the tiles iOS-style, puts a **−** on each
  favourite and a **+** on each catalogue action
- Actions already favourited, and every action once you hit nine, say so via
  toast instead of silently doing nothing

### Categories

Bike, Battery, IOT, Workflow. Inside a tab the category is implied, so labels
drop the qualifier ("Status"); out in favourites and search they revert to the
full name ("Bike status"). Same switch changes the icon from `_tab` to
`_common` art. Full rules in [docs/icons.md](docs/icons.md).

---

## Docs

- **[Interactions](docs/interactions.md)** — every screen, gesture, and state transition
- **[Icon system](docs/icons.md)** — naming rules, `_tab` vs `_common`, adding new icons
- **[Design system](docs/design-system.md)** — tokens, type roles, and component specs as used here

## Known gaps

- Action screens aren't built — tapping one shows a toast
- The Tasks tab is a static screenshot (`assets/tasks-screen.png`); if that file
  is absent, a coded stand-in renders instead
- Favourites live in memory only and reset on reload
