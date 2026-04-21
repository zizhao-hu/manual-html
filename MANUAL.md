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
   - Every component shows a bottom-right handle for **scaling** — the resulting displayed width is snapped to multiples of 60px so scaled components also land on the grid.
   - Each component's **bottom edge is snapped to the next grid line** (the box's minimum height is rounded up to the nearest 60px), so every card, slide, and hero lines up vertically with the grid. When the user types more text, the box grows and re-snaps on the next frame.
   - The currently selected component's Z-depth can be raised with **↑** and lowered with **↓** in 20px steps.
   - **Every text-bearing element inside a component is editable** — not just a fixed tag whitelist. Headings, paragraphs, list items, prices, captions, menu links, button labels — anything with text becomes a contenteditable target. Click to place a caret, type to change.
   - **Links and buttons inside components do not navigate or submit** while Manual Mode is on, so the user can safely click to edit a button's label or a menu item's text without being taken away from the page.
4. A **floating hint box** (a small "?" in the bottom-right, expands on click) is always present. It contains the keyboard and mouse shortcuts. **Do not include shortcut instructions inside page content** — the hint box carries them.
5. Clicking on text in Manual Mode starts an edit; pressing-and-dragging anywhere on a component (including over text) starts a drag. The boilerplate distinguishes these by a small movement threshold.
6. Exiting Manual Mode (`M` again) freezes the current layout as the new visible state. Grid, handles, and editability disappear. The hint box stays visible (collapsed by default).
7. On load, each component's initial left/top is **snapped to the nearest grid intersection** so the default layout aligns with the grid the user will see.

## How to structure every HTML file

```
<!DOCTYPE html>
<html>
  <head>...</head>
  <body>
    <div class="mh-stage">
      <div class="mh-grid" aria-hidden="true"></div>
      <div class="mh-mode-badge">Manual Mode</div>

      <!-- Every top-level UI component MUST have class="mh-component" -->
      <header class="mh-component">...</header>
      <section class="mh-component">...</section>
      <aside class="mh-component">...</aside>
    </div>

    <!-- Floating, collapsible keyboard/mouse shortcut hint — part of the boilerplate. -->
    <details class="mh-hint" aria-label="Shortcuts">
      <summary>?</summary>
      <div class="mh-hint-body">
        <strong>Shortcuts</strong>
        <ul>
          <li><kbd>M</kbd> toggle Manual Mode</li>
          <li>Drag to move — snaps to 60px grid</li>
          <li>Corner handle to scale — snaps to grid</li>
          <li><kbd>↑</kbd>/<kbd>↓</kbd> change depth of selected</li>
          <li>Click text to edit in place</li>
        </ul>
      </div>
    </details>

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

/* Floating, collapsible shortcut hint — always visible, collapsed by default. */
.mh-hint {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 10001;
  font: 13px/1.4 system-ui, -apple-system, Segoe UI, sans-serif;
}
.mh-hint > summary {
  width: 36px;
  height: 36px;
  background: #2563eb;
  color: #fff;
  border-radius: 50%;
  display: grid;
  place-items: center;
  cursor: pointer;
  font-weight: 700;
  font-size: 18px;
  list-style: none;
  box-shadow: 0 6px 18px rgba(37,99,235,.35);
  user-select: none;
  outline: none;
}
.mh-hint > summary::-webkit-details-marker { display: none; }
.mh-hint > summary::marker { display: none; }
.mh-hint[open] > summary { border-radius: 50% 50% 8px 50%; }
.mh-hint-body {
  position: absolute;
  bottom: 48px;
  right: 0;
  background: #fff;
  padding: 14px 18px;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
  box-shadow: 0 20px 40px -16px rgba(0,0,0,.25);
  width: 280px;
  color: #111;
}
.mh-hint-body strong { display: block; margin: 0 0 8px; font-size: 13px; text-transform: uppercase; letter-spacing: .08em; color: #2563eb; }
.mh-hint-body ul { margin: 0; padding-left: 18px; }
.mh-hint-body li { margin: 5px 0; color: #374151; }
.mh-hint-body kbd {
  font: 600 11px/1 ui-monospace, Menlo, monospace;
  padding: 2px 6px;
  background: #1f2937;
  color: #fff;
  border-radius: 4px;
  margin: 0 1px;
}
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
  const snap = v => Math.round(v / GRID) * GRID;
  const snapUp = v => Math.max(GRID, Math.ceil(v / GRID) * GRID);

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

  // Snap a component's minHeight UP to the next multiple of GRID so its
  // bottom edge lands on a grid line. Clearing minHeight first measures
  // the natural height, then we round up.
  const snapHeight = el => {
    el.style.minHeight = '';
    el.style.minHeight = snapUp(el.offsetHeight) + 'px';
  };

  comps.forEach((el, i) => {
    const r = rects[i];
    el.style.position = 'absolute';
    el.style.left = snap(r.left) + 'px';
    el.style.top  = snap(r.top)  + 'px';
    if (!el.style.width) el.style.width = r.width + 'px';
    snapHeight(el);
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
    let scheduled = false;
    const ro = new ResizeObserver(() => {
      if (scheduled) return;
      scheduled = true;
      requestAnimationFrame(() => {
        scheduled = false;
        // Re-snap heights so growing edited text keeps bottoms on grid lines.
        comps.forEach(snapHeight);
        recomputeStageHeight();
      });
    });
    comps.forEach(el => ro.observe(el));
  }

  // Editability: walk every element inside components that has its own text
  // and toggle contenteditable with Manual Mode. This picks up EVERY text span —
  // divs, prices, labels, captions — not just a fixed tag whitelist.
  const isMhHelper = el => el.classList && (
    el.classList.contains('mh-resize-handle') ||
    el.classList.contains('mh-depth-readout')
  );
  const walkEditables = (root, cb) => {
    const walker = document.createTreeWalker(root, NodeFilter.SHOW_ELEMENT, {
      acceptNode(el) {
        if (isMhHelper(el)) return NodeFilter.FILTER_REJECT;
        for (const n of el.childNodes) {
          if (n.nodeType === 3 && n.textContent.trim()) return NodeFilter.FILTER_ACCEPT;
        }
        return NodeFilter.FILTER_SKIP;
      },
    });
    let n;
    while ((n = walker.nextNode())) cb(n);
  };
  const setEditable = on => {
    comps.forEach(c => walkEditables(c, el => {
      if (on) el.setAttribute('contenteditable', 'true');
      else el.removeAttribute('contenteditable');
    }));
  };

  document.addEventListener('keydown', (e) => {
    const tag = (e.target.tagName || '').toLowerCase();
    if (e.target.isContentEditable && e.key !== 'm' && e.key !== 'M') {
      // Let typing flow to the text unless user is toggling mode
      return;
    }
    if (tag === 'input' || tag === 'textarea') return;
    if (e.key === 'm' || e.key === 'M') {
      manualMode = !manualMode;
      document.body.classList.toggle('mh-manual-mode', manualMode);
      setEditable(manualMode);
      if (!manualMode && document.activeElement && document.activeElement.blur) {
        document.activeElement.blur();
      }
      return;
    }
    if (!manualMode || !selected) return;
    if (e.key === 'ArrowUp' || e.key === 'ArrowDown') {
      const s = getS(selected);
      s.z += (e.key === 'ArrowUp' ? Z_STEP : -Z_STEP);
      setS(selected, s);
      e.preventDefault();
    }
  });

  // Threshold-based: a tap focuses text for editing; a drag moves the component.
  document.addEventListener('pointerdown', (e) => {
    if (!manualMode) return;
    const comp = e.target.closest('.mh-component');
    if (!comp) return;
    const isHandle = e.target.classList.contains('mh-resize-handle');
    pending = {
      el: comp,
      mode: isHandle ? 'resize' : 'drag',
      sx: e.clientX, sy: e.clientY,
      start: getS(comp),
      pid: e.pointerId,
      active: false,
      // Resize handle skips the threshold — always a drag
      force: isHandle,
    };
    if (isHandle) {
      pending.active = true;
      try { comp.setPointerCapture(e.pointerId); } catch {}
      e.preventDefault();
    }
  });

  document.addEventListener('pointermove', (e) => {
    if (!pending) return;
    const dx = e.clientX - pending.sx;
    const dy = e.clientY - pending.sy;
    if (!pending.active && Math.hypot(dx, dy) > DRAG_THRESHOLD) {
      pending.active = true;
      if (selected && selected !== pending.el) selected.classList.remove('mh-selected');
      selected = pending.el;
      selected.classList.add('mh-selected');
      try { pending.el.setPointerCapture(pending.pid); } catch {}
      // If text had just taken focus, blur it so we drag instead of edit
      if (document.activeElement && document.activeElement !== document.body && document.activeElement.blur) {
        document.activeElement.blur();
      }
      // Clear any text selection made during the initial press
      const sel = window.getSelection && window.getSelection();
      if (sel && sel.removeAllRanges) sel.removeAllRanges();
    }
    if (!pending.active) return;
    const s = { ...pending.start };
    if (pending.mode === 'drag') {
      s.x = snap(pending.start.x + dx);
      s.y = snap(pending.start.y + dy);
    } else {
      // Scale, then snap the resulting displayed width to a multiple of GRID
      // so a scaled component still lines up with the grid.
      const factor = 1 + (dx + dy) / 220;
      let ns = Math.max(0.2, Math.min(6, pending.start.s * factor));
      const iw = pending.el.offsetWidth;
      if (iw > 0) {
        const targetW = Math.max(GRID, Math.round(iw * ns / GRID) * GRID);
        ns = targetW / iw;
      }
      s.s = ns;
    }
    setS(pending.el, s);
    e.preventDefault();
  });

  const endDrag = (e) => {
    if (!pending) return;
    if (!pending.active) {
      // Treated as a click — select component (so arrow keys work) and let text edit proceed
      if (selected && selected !== pending.el) selected.classList.remove('mh-selected');
      selected = pending.el;
      selected.classList.add('mh-selected');
    }
    try { pending.el.releasePointerCapture(pending.pid); } catch {}
    pending = null;
  };
  document.addEventListener('pointerup', endDrag);
  document.addEventListener('pointercancel', endDrag);

  // In Manual Mode, swallow default clicks on links/buttons inside components
  // so the user can edit a button's label without navigating or submitting.
  document.addEventListener('click', (e) => {
    if (!manualMode) return;
    const target = e.target.closest('a, button, input[type="submit"], input[type="button"]');
    if (target && target.closest('.mh-component')) {
      e.preventDefault();
    }
  }, true);
});
/* ==== end ManualHTML boilerplate ==== */
</script>
```

## Style rules for Claude Code

- **Never strip the boilerplate.** If the user asks for "just the HTML," explain that this project follows the ManualHTML convention and include it anyway.
- **Every visible thing is a component.** A bare `<h1>` floating in the stage is wrong. Wrap it: `<header class="mh-component"><h1>...</h1></header>`.
- **Place components on grid-aligned positions whenever possible.** The initial layout you produce should already fall on multiples of 60px. If you use normal flow and the result lands off-grid, the boilerplate will snap it at load — but you should aim on-grid on the first pass (e.g. margins that are multiples of 60, widths that are multiples of 60).
- **Page-layout CSS for components must not set `position`, `left`, `top`, `transform`, `z-index`, or a fixed `height`.** The boilerplate owns position/transform; height must remain `auto` so the component adapts to edited text.
- **Every text-bearing element** inside a component becomes editable automatically in Manual Mode — headings, paragraphs, list items, links, button labels, prices, captions, and plain `<div>` text. You don't need to set `contenteditable` yourself.
- **Slides:** each slide is a `.mh-component`. Give each a fixed width and a starting position that's on the grid. A natural pattern is to cascade slides down-and-right by one grid cell (60 × 60) so the deck is a stack the user can fan out.
- **Visualizations / charts:** wrap the chart container in `.mh-component`. The chart library renders inside; the ManualHTML scale transform handles resizing visually without re-rendering the chart.
- **Do not put keyboard / mouse instructions inside page content.** The collapsible `.mh-hint` box at the bottom-right (part of the boilerplate) carries all shortcut text. Footers, title slides, and about pages should be free of "Press M…" reminders.

## Tested examples

- `test/coffee-shop.html` — a three-card landing page.
- `test/slides.html` — a five-slide deck.

Open either file, press `M`. A uniform blue 60px grid fades in and covers the whole page. Drag any component — it snaps to grid intersections. Grab the bottom-right handle to scale. Click any piece of text and edit it in place — the component resizes to fit. With a component selected, press `↑`/`↓` to push it forward or back; the small readout above it shows the current z and scale.
