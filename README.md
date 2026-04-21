# ManualHTML

A tiny convention that turns any HTML Claude Code generates into a spatial, editable canvas. Every UI component lives on a 3D grid — drag it, scale it, push it forward or back.

## What it is

A single file: [`MANUAL.md`](./MANUAL.md). Drop it into any repo, tell Claude Code to follow it, and every HTML page, slide deck, or visualization it generates will come with:

- A 3D grid (X/Y position, Z depth)
- Draggable, scalable components
- `↑` / `↓` to change the selected component's depth
- `M` to toggle **Move Mode** — the grid appears, components become editable
- Outside Move Mode the page renders like any normal site

## Use

```
> Read MANUAL.md and follow it. Build me a landing page for a coffee shop.
```

or, for recurring use, drop `MANUAL.md` at the root of your project and reference it in your `CLAUDE.md`:

```
When generating HTML of any kind, read and follow ./MANUAL.md.
```

## Test

[`test/coffee-shop.html`](./test/coffee-shop.html) is a synthetic deliverable produced by following `MANUAL.md`. Open it in a browser:

1. It looks like a normal coffee shop landing page.
2. Press `M`. A blue grid fades in and a "Move Mode" badge appears.
3. Click a card, drag it around. Grab its bottom-right blue handle to scale.
4. With a card selected, press `↑` a few times — it rises toward you (Z depth).
5. Press `M` again — the grid disappears and the new arrangement sticks.

## Why

LLM-generated HTML is usually frozen on arrival. ManualHTML gives every output a built-in editing surface so the user can manually reflow what the model produced without touching code.

## License

MIT — see [`LICENSE`](./LICENSE).
