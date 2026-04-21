# ManualHTML

Convention for Claude Code: when the user asks for HTML (page, deck, dashboard, viz, anything), follow this spec.

## Core rule

Every UI component lives on a 3D grid. X/Y snap to a uniform **60px** grid. Z (depth) is controlled with `↑`/`↓` in **20px** steps. Components are draggable, scalable, and every text element inside them is directly editable. Grid, handles, and editability are visible only in **Manual Mode** (toggled with `M`).

## Required behavior

1. **Default view**: normal page — no grid, components static, text not editable.
2. **`M` toggles Manual Mode.** In Manual Mode:
   - A uniform 60×60 px grid overlay spans the stage, all the way to the bottom.
   - A "Manual Mode" badge appears top-right.
   - Every `.mh-component` is draggable (snaps X/Y to 60px) and has a bottom-right scale handle (resulting displayed width snaps to multiples of 60).
   - Each component's **bottom snaps up** to the next grid line — `min-height` is rounded up to the nearest 60px, and re-snaps on the next frame when text grows.
   - Selected component's Z-depth: `↑`/`↓` in 20px steps.
   - **Every text-bearing element inside a component becomes editable** — headings, paragraphs, list items, prices, captions, menu links, button labels, plain `<div>` text. No whitelist.
   - **Links and buttons do not navigate or submit** while Manual Mode is on, so the user can click into button/link text to edit it.
3. **Floating `?` hint box** (collapsible, bottom-right) is always present and carries all shortcut text. **Do not put shortcut instructions inside page content.**
4. Click text → edit; press-and-drag → move. Distinguished by a 4px movement threshold.
5. Exit Manual Mode (`M` again): freezes layout, grid/handles/editability disappear, hint stays (collapsed).
6. On load, each component's initial left/top is snapped to the nearest grid intersection.

## File skeleton

```html
<!DOCTYPE html>
<html>
  <head>...</head>
  <body>
    <div class="mh-stage">
      <div class="mh-grid" aria-hidden="true"></div>
      <div class="mh-mode-badge">Manual Mode</div>

      <!-- Every top-level UI block MUST have class="mh-component" -->
      <header class="mh-component">...</header>
      <section class="mh-component">...</section>
    </div>

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

    <style>/* boilerplate + page styles */</style>
    <script>/* boilerplate */</script>
  </body>
</html>
```

Everything visible goes inside `.mh-stage`. Every logical block (nav, hero, card, chart, slide, footer) gets `class="mh-component"` on its outer wrapper.

## Boilerplate CSS — paste verbatim; page styles come after

```html
<style>
/* ==== ManualHTML boilerplate ==== */
html, body { margin: 0; padding: 0; }
body { font: 16px/1.5 system-ui, -apple-system, Segoe UI, sans-serif; background: #f7f7f8; color: #111; }

.mh-stage {
  position: relative;
  perspective: 1600px;
  transform-style: preserve-3d;
  min-height: 100vh;
  /* No overflow:hidden — grid/components must extend with content. */
}

.mh-component {
  /* Start in flow so natural layout is measured; JS swaps to absolute post-measure. */
  position: relative;
  transform-style: preserve-3d;
  transform-origin: top left;
  outline: 0 solid transparent;
  transition: outline-color .12s;
  touch-action: none;
  height: auto;
  box-sizing: border-box;
}

.mh-resize-handle {
  position: absolute; right: -7px; bottom: -7px;
  width: 14px; height: 14px;
  background: #2563eb; border: 2px solid #fff; border-radius: 50%;
  cursor: nwse-resize; display: none; z-index: 5;
  box-shadow: 0 2px 6px rgba(0,0,0,.25);
}
body.mh-manual-mode .mh-resize-handle { display: block; }
body.mh-manual-mode .mh-component { outline: 1px dashed rgba(37,99,235,.55); cursor: grab; }
body.mh-manual-mode .mh-component.mh-selected { outline: 2px solid #2563eb; }
body.mh-manual-mode .mh-component:active { cursor: grabbing; }
body.mh-manual-mode .mh-component [contenteditable="true"] { cursor: text; outline: none; }
body.mh-manual-mode .mh-component [contenteditable="true"]:focus {
  background: rgba(37,99,235,.08);
  border-radius: 3px;
  box-shadow: 0 0 0 2px rgba(37,99,235,.25);
}

.mh-grid {
  position: absolute; inset: 0;
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
  position: fixed; top: 12px; right: 12px;
  padding: 8px 14px;
  background: #2563eb; color: #fff;
  border-radius: 999px;
  font: 600 12px/1.3 system-ui, sans-serif;
  z-index: 10000;
  display: none;
  box-shadow: 0 4px 12px rgba(37,99,235,.35);
  max-width: 520px;
}
body.mh-manual-mode .mh-mode-badge { display: block; }

.mh-depth-readout {
  position: absolute; top: -22px; left: 0;
  font: 600 11px/1 ui-monospace, Menlo, monospace;
  color: #2563eb; background: #fff;
  padding: 2px 6px; border-radius: 4px;
  border: 1px solid rgba(37,99,235,.4);
  pointer-events: none; display: none;
}
body.mh-manual-mode .mh-component.mh-selected .mh-depth-readout { display: block; }

.mh-hint {
  position: fixed; bottom: 20px; right: 20px;
  z-index: 10001;
  font: 13px/1.4 system-ui, -apple-system, Segoe UI, sans-serif;
}
.mh-hint > summary {
  width: 36px; height: 36px;
  background: #2563eb; color: #fff;
  border-radius: 50%;
  display: grid; place-items: center;
  cursor: pointer;
  font-weight: 700; font-size: 18px;
  list-style: none;
  box-shadow: 0 6px 18px rgba(37,99,235,.35);
  user-select: none; outline: none;
}
.mh-hint > summary::-webkit-details-marker { display: none; }
.mh-hint > summary::marker { display: none; }
.mh-hint[open] > summary { border-radius: 50% 50% 8px 50%; }
.mh-hint-body {
  position: absolute; bottom: 48px; right: 0;
  background: #fff;
  padding: 14px 18px;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
  box-shadow: 0 20px 40px -16px rgba(0,0,0,.25);
  width: 280px; color: #111;
}
.mh-hint-body strong { display: block; margin: 0 0 8px; font-size: 13px; text-transform: uppercase; letter-spacing: .08em; color: #2563eb; }
.mh-hint-body ul { margin: 0; padding-left: 18px; }
.mh-hint-body li { margin: 5px 0; color: #374151; }
.mh-hint-body kbd {
  font: 600 11px/1 ui-monospace, Menlo, monospace;
  padding: 2px 6px;
  background: #1f2937; color: #fff;
  border-radius: 4px;
  margin: 0 1px;
}
/* ==== end boilerplate ==== */
</style>
```

## Boilerplate JS — paste verbatim before `</body>`

```html
<script>
/* ==== ManualHTML boilerplate ==== */
document.addEventListener('DOMContentLoaded', function () {
  const GRID = 60, Z_STEP = 20, DRAG_THRESHOLD = 4;
  const snap = v => Math.round(v / GRID) * GRID;
  const snapUp = v => Math.max(GRID, Math.ceil(v / GRID) * GRID);

  const stage = document.querySelector('.mh-stage');
  if (!stage) return;
  let manualMode = false, selected = null, pending = null;

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

  // Measure natural positions, then lock each component to snapped absolute.
  const comps = Array.from(document.querySelectorAll('.mh-component'));
  const sr = stage.getBoundingClientRect();
  const rects = comps.map(el => {
    const r = el.getBoundingClientRect();
    return { left: r.left - sr.left, top: r.top - sr.top, width: r.width };
  });

  // Snap minHeight UP so each component's bottom lands on a grid line.
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
    const h = document.createElement('div'); h.className = 'mh-resize-handle'; el.appendChild(h);
    const d = document.createElement('div'); d.className = 'mh-depth-readout';
    d.textContent = 'z: 0  ·  scale: 1.00'; el.appendChild(d);
  });

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
        comps.forEach(snapHeight);
        recomputeStageHeight();
      });
    });
    comps.forEach(el => ro.observe(el));
  }

  // Walk every element with its own text and toggle contenteditable — no whitelist.
  const isMhHelper = el => el.classList && (
    el.classList.contains('mh-resize-handle') || el.classList.contains('mh-depth-readout')
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
    let n; while ((n = walker.nextNode())) cb(n);
  };
  const setEditable = on => {
    comps.forEach(c => walkEditables(c, el => {
      if (on) el.setAttribute('contenteditable', 'true');
      else el.removeAttribute('contenteditable');
    }));
  };

  document.addEventListener('keydown', (e) => {
    const tag = (e.target.tagName || '').toLowerCase();
    if (e.target.isContentEditable && e.key !== 'm' && e.key !== 'M') return;
    if (tag === 'input' || tag === 'textarea') return;
    if (e.key === 'm' || e.key === 'M') {
      manualMode = !manualMode;
      document.body.classList.toggle('mh-manual-mode', manualMode);
      setEditable(manualMode);
      if (!manualMode && document.activeElement && document.activeElement.blur) document.activeElement.blur();
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

  // Tap = edit text; drag = move. 4px threshold separates them.
  document.addEventListener('pointerdown', (e) => {
    if (!manualMode) return;
    const comp = e.target.closest('.mh-component');
    if (!comp) return;
    const isHandle = e.target.classList.contains('mh-resize-handle');
    pending = {
      el: comp, mode: isHandle ? 'resize' : 'drag',
      sx: e.clientX, sy: e.clientY,
      start: getS(comp), pid: e.pointerId,
      active: false, force: isHandle,
    };
    if (isHandle) {
      pending.active = true;
      try { comp.setPointerCapture(e.pointerId); } catch {}
      e.preventDefault();
    }
  });

  document.addEventListener('pointermove', (e) => {
    if (!pending) return;
    const dx = e.clientX - pending.sx, dy = e.clientY - pending.sy;
    if (!pending.active && Math.hypot(dx, dy) > DRAG_THRESHOLD) {
      pending.active = true;
      if (selected && selected !== pending.el) selected.classList.remove('mh-selected');
      selected = pending.el;
      selected.classList.add('mh-selected');
      try { pending.el.setPointerCapture(pending.pid); } catch {}
      if (document.activeElement && document.activeElement !== document.body && document.activeElement.blur) document.activeElement.blur();
      const sel = window.getSelection && window.getSelection();
      if (sel && sel.removeAllRanges) sel.removeAllRanges();
    }
    if (!pending.active) return;
    const s = { ...pending.start };
    if (pending.mode === 'drag') {
      s.x = snap(pending.start.x + dx);
      s.y = snap(pending.start.y + dy);
    } else {
      // Scale, then snap resulting displayed width to a multiple of GRID.
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

  const endDrag = () => {
    if (!pending) return;
    if (!pending.active) {
      if (selected && selected !== pending.el) selected.classList.remove('mh-selected');
      selected = pending.el;
      selected.classList.add('mh-selected');
    }
    try { pending.el.releasePointerCapture(pending.pid); } catch {}
    pending = null;
  };
  document.addEventListener('pointerup', endDrag);
  document.addEventListener('pointercancel', endDrag);

  // In Manual Mode, swallow clicks on links/buttons inside components so the user can edit their text.
  document.addEventListener('click', (e) => {
    if (!manualMode) return;
    const target = e.target.closest('a, button, input[type="submit"], input[type="button"]');
    if (target && target.closest('.mh-component')) e.preventDefault();
  }, true);
});
/* ==== end boilerplate ==== */
</script>
```

## Style rules

- Never strip the boilerplate. If the user asks for "just the HTML," include it anyway — it's the convention.
- Every visible block is a component. Wrap a bare `<h1>` in `<header class="mh-component">...</header>`.
- Aim on-grid on first pass: margins and widths that are multiples of 60px. The boilerplate will snap anything off-grid at load, but on-grid input gives the cleanest default.
- Page CSS for components must **not** set `position`, `left`, `top`, `transform`, `z-index`, or fixed `height`. Boilerplate owns position/transform; `height: auto` lets text grow.
- Every text element becomes editable automatically in Manual Mode — don't set `contenteditable` yourself.
- Slides: one `.mh-component` per slide, fixed width, starting positions on-grid (cascading by 60×60 per slide works well).
- Charts/visualizations: wrap the container in `.mh-component`. The scale transform handles visual resizing without re-rendering.
- Do **not** put keyboard/mouse instructions in page content — the `.mh-hint` box carries them.

## Examples in this repo

- `test/coffee-shop.html` — three-card landing page.
- `test/slides.html` — five-slide deck.

Open either, press `M`: blue 60px grid fades in, drag components (snap to grid), corner-handle scales (width snaps to 60), click text to edit, `↑`/`↓` change depth of the selected component.
