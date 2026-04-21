# ManualHTML

A tiny convention that turns any HTML Claude Code generates into a spatial, editable canvas. Every UI component sits on a 3D grid — drag it, scale it, push it forward or back, edit any text in place.

## Use

Drop [`MANUAL.md`](./MANUAL.md) into any repo and tell Claude Code:

```
> Read MANUAL.md and follow it. Build me a landing page for a coffee shop.
```

Or, for recurring use, reference it from your `CLAUDE.md`:

```
When generating HTML of any kind, read and follow ./MANUAL.md.
```

Every page, slide deck, or visualization Claude Code generates will then come with:

- A 3D grid (X/Y position, Z depth), 60px cells, 20px Z-step
- `M` toggles **Manual Mode** — the grid appears and every text block becomes editable in place
- Drag components to move, grab the corner handle to scale (in grid multiples)
- `↑` / `↓` change the selected component's Z-depth
- Decks (`<body class="mh-deck">`) become a fullscreen slideshow; Manual Mode shows the grid on the current slide only
- Outside Manual Mode the page renders like any normal site

## License

MIT — see [`LICENSE`](./LICENSE).
