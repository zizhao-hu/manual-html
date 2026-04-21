# ManualHTML

A tiny convention that turns any HTML Claude Code generates into a spatial, editable canvas. Every UI component lives on a 3D grid — drag it, scale it, push it forward or back, edit any text in place.

## What it is

A single file: [`MANUAL.md`](./MANUAL.md). Drop it into any repo, tell Claude Code to follow it, and every HTML page, slide deck, or visualization it generates will come with:

- A 3D grid (X/Y position, Z depth), 60px cells, 20px Z-step
- Draggable, scalable components — corner handle scales in grid multiples
- `↑` / `↓` change the selected component's Z-depth
- `M` toggles **Manual Mode** — the grid appears and every text block becomes editable in place
- Outside Manual Mode the page renders like any normal site
- Decks (`<body class="mh-deck">`) become a fullscreen slideshow; Manual Mode shows the grid on the current slide only

## Use

```
> Read MANUAL.md and follow it. Build me a landing page for a coffee shop.
```

or, for recurring use, drop `MANUAL.md` at the root of your project and reference it in your `CLAUDE.md`:

```
When generating HTML of any kind, read and follow ./MANUAL.md.
```

## Test

Two synthetic deliverables produced by following `MANUAL.md`:

### [`test/coffee-shop.html`](./test/coffee-shop.html) — a landing page
1. It looks like a normal coffee shop landing page.
2. Press `M`. A blue grid fades in and a "Manual Mode" badge appears.
3. Click a card, drag it around. Grab its bottom-right blue handle to scale.
4. With a card selected, press `↑` a few times — it rises toward you (Z depth).
5. Click any heading or paragraph to edit the text in place.
6. Press `M` again — 