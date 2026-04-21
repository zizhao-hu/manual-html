# ManualHTML

> A convention for Claude Code. When the user asks you to generate HTML — a webpage, slide deck, data visualization, landing page, dashboard, or any other UI output — follow this manual.

---

## Core rule

**Every UI component lives on a 3D grid.** X and Y are screen position, laid out on a **uniform 60px grid** that covers the entire stage. Z is depth (front ↔ back), controlled with the Up/Down arrow keys. Components are draggable, scalable, and their text is directly editable. Every drag **snaps** the component to the nearest 60px grid intersection so layouts stay aligned. A grid overlay is shown while the user is in **Manual Mode** and hidden otherwise.

## Required behavior

When a user opens a page you generate, it must:

1. Render like a normal page by default (no visible grid, components static, text not editable).
2. Toggle **Manual Mode** when the user presses `M`.
3. In Manual Mode:
   - A single, uniform 60 × 60 px grid overlay becomes visible across the entire stage — squares equal everywhere, visible all the way to the bottom of the content.
   - A small "Manual Mode" badge appears in the top-right.
   - Every top-level UI component becomes **draggable** with the mouse/pointer, **snapping** to 60px grid cells.
   - Every component shows a bottom-right handle for **scaling**.
   - The currently selected component's Z-depth can be raised with **↑** and lowered with **↓** in 20px steps.
   - **All text inside components is editable** (click to place a caret, type to change it). When text content grows or shrinks, the component **adapts** — its box resizes to fit the new content.
4. Clicking on text in Manual Mode starts an edit; pressing-and-dragging anywhere on a component (including over text) starts a drag. The boilerplate distinguishes these by a small movement threshold.
5. Exiting Manual Mode (`M` again) freezes the current layout as the new visible state. Grid, handles, and editability disappear.
6. On load, each component's initial left/top is **snapped to the nearest grid intersection** so the default layout aligns with the grid the user will see.

## How to structure every HTML file

```
<!DOCTYPE html>
<html>
  <head>...</head>
  <body>
    <div class="mh-stage">
      <div class="mh-grid" aria-hidden="true"></div>
      <div class="mh-mode-badge">Manual Mode — drag (snaps), corner = scale, ↑/↓ = depth, click text to edit</div>

      <!-- Every top-level UI component MUST have class="mh-component" -->
      <header class="mh-component">...</header>
      <section class="mh-component">...</section>
      <aside class="mh-component">...</aside>
    </div>

    <style>/* ManualHTML boilerplate + page styles */</style>
    <script>/* ManualHTML boilerplate */</script>
  </body>
</html>
```

Everything the user wants to see must be placed inside `.mh-stage`. Each logical component — a nav bar, a hero, a product card, a chart, a slide, a footer — gets `class="mh-component"` on its outer wrapper.

## Boilerplate — copy into every file verbatim

Paste the following CSS block. Page-specific styles go **after** it so they can override padding/colors but not the positioning model.

```html
<style>
/* ==== ManualHTML boilerplate (do not modify) ==== */
html, body { margin: 0; padding: 0; }
body { font: 16px/1.5 system-ui, -apple-system, Segoe UI, sans-serif; background: #f7f7f8; color: #111; }

.mh-stage {
  position: relative;
  perspective: 1600px;
  transform-style: preserve-3d;
  min-height: 100vh;
  /* NO overflow: hidden — the grid and components must extend down with the content */
}

.mh-component {
  /* Start in normal flow so measurement at load reflects the intended layout.
     The boilerplate JS switches each component to `position: absolute` after
     measuring. If this were `absolute` from the start, every component's
     "static position" would collapse toward the top-left of the stage and
     they would stack in a corner. */
  position: relative;
  transform-style: preserve-3d;
  transform-origin: top left;
  outline: 0 solid transparent;
  transition: outline-color .12s;
  touch-action: none;
  /* Components adapt to text: width is fixed at init, height grows with content */
  height: auto;
  box-sizing: border-box;
}

.mh-resize-handle {
  position: absolute;
  right: -7px;
  bottom: -7px;
  width: 14px;
  height: 14px;
  background: #2563eb;
  border: 2px solid #fff;
  border-radius: 50%;
  cursor: nwse-resize;
  display: none;
  z-index: 5;
  box-shadow: 0 2px 6px rgba(0,0,0,.25);
}
body.mh-manual-mode .mh-resize-handle { display: block; }
body.mh-manual-mode .mh-component { outline: 1px dashed rgba(37, 99, 235, .55); cursor: grab; }
body.mh-manual-mode .mh-component.mh-selected { outline: 2px solid #2563eb; }
body.mh-manual-mode .mh-component:active { cursor: grabbing; }
body.mh-manual-mode .mh-component [contenteditable="true"] { cursor: text; outline: none; }
body.mh-manual-mode .mh-component [contenteditable="true"]:focus {
  background: rgba(37,99,235,.08);
  border-radius: 3px;
  box-shadow: 0 0 0 2px rgba(37,99,235,.25);
}

/* Single uniform grid, stage-wide, 60px cells, top-left origin. */
.mh-grid {
  position: absolute;
  inset: 0;
  pointer-events: none;
  opacity: 0;
  transition: opacity .2s;
  z-index: 0;
  background-image:
    linear-gradient(rgba(37,99,235,.32) 1px, transparent 1px),
    linear-gradient(90deg, rgba(37,99,235,.32) 1px, transparent 1px);
  background-size: 60px 60px;
  background-position: 0 0;
}
body.mh-manual-mode .mh-grid { opacity: 1; }

.mh-mode-badge {
  position: fixed;
  top: 12px;
  right: 12px;
  padding: 8px 14px;
  background: #2563eb;
  color: #fff;
  border-radius: 999px;
  font: 600 12px/1.3 system-ui, sans-serif;
  z-index: 10000;
  display: none;
  box-shadow: 0 4px 12px rgba(37,99,235,.35);
  max-width: 520px;
}
body.mh-manual-mode .mh-mode-badge { display: block; }

.mh-depth-readout {
  position: absolute;
  top: -22px;
  left: 0;
  font: 600 11px/1 ui-monospace, Menlo, monospace;
  color: #2563eb;
  background: #fff;
  padding: 2px 6px;
  border-radius: 4px;
  border: 1px solid rgba(37,99,235,.4);
  pointer-events: none;
  display: none;
}
body.mh-manual-mode .mh-component.mh-selected .mh-depth-readout { display: block; }
/* ==== end ManualHTML boilerplate ==== */
</style>
```

Paste the following script just before `</body>`.

```html
<script>
/* ==== ManualHTML boilerplate (do not modify) ==== */
document.addEventListener('DOMContentLoaded', function () {
  const GRID = 60;
  const Z_STEP = 20;
  const DRAG_THRESHOLD = 4;  // px — below this a pointerdown is treated as a click (text edit)
  const EDITABLE_SEL = 'h1,h2,h3,h4,h5,h6,p,span,a,li,blockquote,figcaption,dt,dd,th,td,strong,em';
  const snap = v => Math.round(v / GRID) * GRID;

  const stage = document.querySelector('.mh-stage');
  if (!stage) return;
  let manualMode = false;
  let selected = null;
  let pending = null;

  const getS = el => ({
    x: parseFloat(el.dataset.mhX || '0'),
    y: parseFloat(el.dataset.mhY || '0'),
    z: parseFloat(el.dataset.mhZ || '0'),
    s: parseFloat(el.dataset.mhScale || '1'),
  });
  const setS = (el, s) => {
    el.dataset.mhX = s.x; el.dataset.mhY = s.y; el.dataset.mhZ = s.z; el.dataset.mhScale = s.s;
    el.style.transform = `translate3d(${s.x}px, ${s.y}px, ${s.z}px) scale(${s.s})`;
    el.style.zIndex = String(1000 + Math.round(s.z));
    const r = el.querySelector(':scope > .mh-depth-readout');
    if (r) r.textContent = `z: ${Math.round(s.z)}  ·  scale: ${s.s.toFixed(2)}`;
  };

  // Measure all natural positions first, then lock everything to snapped absolute.
  const comps = Array.from(document.querySelectorAll('.mh-component'));
  const sr = stage.getBoundingClientRect();
  const rects = comps.map(el => {
    const r = el.getBoundingClientRect();
    return { left: r.left - sr.left, top: r.top - sr.top, width: r.width };
  });

  comps.forEach((el, i) => {
    const r = rects[i];
    el.style.position = 'absolute';
    el.style.left = snap(r.left) + 'px';
    el.style.top  = snap(r.top)  + 'px';
    if (!el.style.width) el.style.width = r.width + 'px';
    setS(el, getS(el));

    const h = document.createElement('div');
    h.className = 'mh-resize-handle';
    el.appendChild(h);

    const d = document.createElement('div');
    d.className = 'mh-depth-readout';
    d.textContent = 'z: 0  ·  scale: 1.00';
    el.appendChild(d);
  });

  // Stage height adapts to whatever's inside — recompute on any component resize
  // (so growing text keeps the grid underneath it and nothing gets clipped).
  const recomputeStageHeight = () => {
    let mb = 0;
    comps.forEach(el => {
      const top = parseFloat(el.style.top || '0');
      mb = Math.max(mb, top + el.offsetHeight);
    });
    stage.style.minHeight = Math.max(mb + 120, window.innerHeight) + 'px';
  };
  recomputeStageHeight();
  if (window.ResizeObserver) {
    const ro = new ResizeObserver(recomputeStageHeight);
    comps.forEach(el => ro.observe(el));
  }

  // Editability: turn text elements inside components on/off with Manual Mode.
  const editables = comps.flatMap(c => Array.from(c.querySelectorAll(EDITABLE_SEL)));
  const setEditable = on => {
    editables.forEach(el => {
      if (on) el.setAttribute('contenteditable', 'true');
      else el.removeAttribute('contenteditable');
    });
  };

  document.addEventListener('keydown', (e) => {
    const tag = (e.target.tagName || '').toLowerCase();
    if (e.target.isContentEditable && e.key !== 'm' && e.key !== 'M') {
      // Let typing flow to the text unless user is toggling mode
      return;
    }
    if (tag === 'input' |