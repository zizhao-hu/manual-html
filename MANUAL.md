# ManualHTML

Every HTML output is wired with a 3D-grid editable layer. `M` toggles **Manual Mode** — 60px grid appears, components become draggable/scalable, every text element becomes editable in place. `↑`/`↓` change the selected component's Z in 20px steps.

**Scope: behavior only.** Visual style — colors, typography, layout, imagery, aesthetic — is the user's to shape. This spec has no opinion on any of it.

## Rules

1. **Every atomic block is its own `.mh-component`.** Row of three cards → three components. Stats strip of four numbers → four. Testimonial grid → one per quote. If moving one should leave the others behind, they're separate.
2. **Don't set `position`, `left`, `top`, `transform`, `z-index`, or fixed `height` on `.mh-component`** — the boilerplate owns those. `height: auto` so text grows.
3. **Don't set `contenteditable` yourself** — Manual Mode handles it.
4. **Keyboard shortcuts live in the `.mh-hint` box only** — not in page content.
5. **Don't strip or rename the boilerplate classes.** `.mh-stage`, `.mh-component`, `.mh-slide`, `.mh-grid`, `.mh-mode-badge`, `.mh-hint` are the contract.

## Skeleton

```html
<!DOCTYPE html>
<html>
<head><style>/* boilerplate + your page styles */</style></head>
<body>
  <div class="mh-stage">
    <div class="mh-grid" aria-hidden="true"></div>
    <div class="mh-mode-badge">Manual Mode</div>
    <!-- every atomic block gets class="mh-component" -->
    <header class="mh-component">…</header>
    <section class="mh-component">…</section>
  </div>
  <details class="mh-hint"><summary>?</summary>
    <div class="mh-hint-body">
      <strong>Shortcuts</strong>
      <ul>
        <li><kbd>M</kbd> toggle Manual Mode</li>
        <li>Drag to move, corner handle to scale — snaps to 60px</li>
        <li><kbd>↑</kbd>/<kbd>↓</kbd> depth of selected</li>
        <li>Click text to edit</li>
      </ul>
    </div>
  </details>
  <script>/* boilerplate */</script>
</body>
</html>
```

## Deck mode

Add `class="mh-deck"` to `<body>`. **Slides are containers, not components** — components live *inside* slides.

```html
<body class="mh-deck">
  <div class="mh-stage">
    <section class="mh-slide">
      <div class="mh-component">title</div>
      <div class="mh-component">body paragraph</div>
      <div class="mh-component">bullets</div>
    </section>
    <section class="mh-slide">…</section>
  </div>
</body>
```

Default renders as a fullscreen slideshow: `←`/`→`/Space/click advance, counter bottom-left. Manual Mode shows the grid on the **current slide only**; only that slide's components are editable; `←`/`→` still navigate so you can edit each slide in turn. No unfold.

## Boilerplate CSS — paste verbatim

Theme the Manual-Mode chrome by overriding `--mh-accent` (and the other `--mh-*` vars) in your own CSS.

```html
<style>
html,body{margin:0;padding:0}
:root{--mh-accent:#2563eb;--mh-grid-line:rgba(37,99,235,.32);--mh-hint-bg:#fff;--mh-hint-fg:#111;--mh-hint-border:rgba(0,0,0,.1)}
.mh-stage{position:relative;perspective:1600px;transform-style:preserve-3d;min-height:100vh}
.mh-component{position:relative;transform-style:preserve-3d;transform-origin:top left;outline:0 solid transparent;transition:outline-color .12s;touch-action:none;height:auto;box-sizing:border-box}
.mh-resize-handle{position:absolute;right:-7px;bottom:-7px;width:14px;height:14px;background:var(--mh-accent);border:2px solid #fff;border-radius:50%;cursor:nwse-resize;display:none;z-index:5;box-shadow:0 2px 6px rgba(0,0,0,.25)}
body.mh-manual-mode .mh-resize-handle{display:block}
body.mh-manual-mode .mh-component{outline:1px dashed color-mix(in srgb,var(--mh-accent) 55%,transparent);cursor:grab}
body.mh-manual-mode .mh-component.mh-selected{outline:2px solid var(--mh-accent)}
body.mh-manual-mode .mh-component:active{cursor:grabbing}
body.mh-manual-mode .mh-component [contenteditable="true"]{cursor:text;outline:none}
body.mh-manual-mode .mh-component [contenteditable="true"]:focus{background:color-mix(in srgb,var(--mh-accent) 8%,transparent);border-radius:3px;box-shadow:0 0 0 2px color-mix(in srgb,var(--mh-accent) 25%,transparent)}
.mh-grid{position:absolute;inset:0;pointer-events:none;opacity:0;transition:opacity .2s;z-index:0;background-image:linear-gradient(var(--mh-grid-line) 1px,transparent 1px),linear-gradient(90deg,var(--mh-grid-line) 1px,transparent 1px);background-size:60px 60px}
body.mh-manual-mode .mh-grid{opacity:1}
.mh-mode-badge{position:fixed;top:12px;right:12px;padding:8px 14px;background:var(--mh-accent);color:#fff;border-radius:999px;font:600 12px/1.3 system-ui,sans-serif;z-index:10000;display:none;box-shadow:0 4px 12px rgba(0,0,0,.2)}
body.mh-manual-mode .mh-mode-badge{display:block}
.mh-depth-readout{position:absolute;top:-22px;left:0;font:600 11px/1 ui-monospace,Menlo,monospace;color:var(--mh-accent);background:#fff;padding:2px 6px;border-radius:4px;border:1px solid color-mix(in srgb,var(--mh-accent) 40%,transparent);pointer-events:none;display:none}
body.mh-manual-mode .mh-component.mh-selected .mh-depth-readout{display:block}
.mh-hint{position:fixed;bottom:20px;right:20px;z-index:10001;font:13px/1.4 system-ui,sans-serif}
.mh-hint>summary{width:36px;height:36px;background:var(--mh-accent);color:#fff;border-radius:50%;display:grid;place-items:center;cursor:pointer;font-weight:700;font-size:18px;list-style:none;box-shadow:0 6px 18px rgba(0,0,0,.25);user-select:none;outline:none}
.mh-hint>summary::-webkit-details-marker,.mh-hint>summary::marker{display:none}
.mh-hint[open]>summary{border-radius:50% 50% 8px 50%}
.mh-hint-body{position:absolute;bottom:48px;right:0;background:var(--mh-hint-bg);padding:14px 18px;border-radius:12px;border:1px solid var(--mh-hint-border);box-shadow:0 20px 40px -16px rgba(0,0,0,.25);width:280px;color:var(--mh-hint-fg)}
.mh-hint-body strong{display:block;margin:0 0 8px;font-size:13px;text-transform:uppercase;letter-spacing:.08em;color:var(--mh-accent)}
.mh-hint-body ul{margin:0;padding-left:18px}
.mh-hint-body li{margin:5px 0}
.mh-hint-body kbd{font:600 11px/1 ui-monospace,Menlo,monospace;padding:2px 6px;background:#1f2937;color:#fff;border-radius:4px;margin:0 1px}
body.mh-deck .mh-slide{position:relative}
body.mh-deck .mh-grid{display:none}
body.mh-deck.mh-deck-ready .mh-stage{position:static;min-height:0;perspective:none;transform-style:flat}
body.mh-deck.mh-deck-ready .mh-slide{position:fixed!important;inset:0!important;width:100vw!important;height:100vh!important;margin:0!important;box-sizing:border-box;overflow:hidden;display:none!important;z-index:5000}
body.mh-deck.mh-deck-ready .mh-slide.mh-current{display:block!important}
body.mh-deck.mh-deck-ready.mh-manual-mode .mh-slide.mh-current::before{content:'';position:absolute;inset:0;pointer-events:none;z-index:0;background-image:linear-gradient(var(--mh-grid-line) 1px,transparent 1px),linear-gradient(90deg,var(--mh-grid-line) 1px,transparent 1px);background-size:60px 60px}
body.mh-deck.mh-deck-ready:not(.mh-manual-mode){cursor:pointer}
body.mh-deck.mh-deck-ready:not(.mh-manual-mode) [contenteditable],body.mh-deck.mh-deck-ready:not(.mh-manual-mode) a,body.mh-deck.mh-deck-ready:not(.mh-manual-mode) button,body.mh-deck.mh-deck-ready:not(.mh-manual-mode) .mh-hint{cursor:auto}
.mh-deck-counter{position:fixed;bottom:20px;left:20px;padding:6px 12px;background:rgba(17,24,39,.72);color:#fff;border-radius:999px;font:600 12px/1 ui-monospace,Menlo,monospace;z-index:10002;display:none;pointer-events:none}
body.mh-deck.mh-deck-ready .mh-deck-counter{display:block}
</style>
```

## Boilerplate JS — paste verbatim before `</body>`

```html
<script>
document.addEventListener('DOMContentLoaded',function(){
const GRID=60,Z=20,DT=4;
const snap=v=>Math.round(v/GRID)*GRID,snapUp=v=>Math.max(GRID,Math.ceil(v/GRID)*GRID);
const stage=document.querySelector('.mh-stage');if(!stage)return;
let mm=false,sel=null,pd=null;
const isDeck=document.body.classList.contains('mh-deck');
const slides=isDeck?Array.from(document.querySelectorAll('.mh-slide')):[];
const getS=e=>({x:+(e.dataset.mhX||0),y:+(e.dataset.mhY||0),z:+(e.dataset.mhZ||0),s:+(e.dataset.mhScale||1)});
const setS=(e,s)=>{e.dataset.mhX=s.x;e.dataset.mhY=s.y;e.dataset.mhZ=s.z;e.dataset.mhScale=s.s;e.style.transform=`translate3d(${s.x}px,${s.y}px,${s.z}px) scale(${s.s})`;e.style.zIndex=String(1000+Math.round(s.z));const r=e.querySelector(':scope > .mh-depth-readout');if(r)r.textContent=`z: ${Math.round(s.z)}  ·  scale: ${s.s.toFixed(2)}`};
const comps=Array.from(document.querySelectorAll('.mh-component'));
const cont=e=>(isDeck&&e.closest('.mh-slide'))||stage;
const rects=comps.map(e=>{const c=cont(e),cr=c.getBoundingClientRect(),r=e.getBoundingClientRect();return{left:r.left-cr.left,top:r.top-cr.top,w:r.width}});
const snapH=e=>{e.style.minHeight='';e.style.minHeight=snapUp(e.offsetHeight)+'px'};
comps.forEach((e,i)=>{const r=rects[i];e.style.position='absolute';e.style.left=snap(r.left)+'px';e.style.top=snap(r.top)+'px';if(!e.style.width)e.style.width=r.w+'px';snapH(e);setS(e,getS(e));const h=document.createElement('div');h.className='mh-resize-handle';e.appendChild(h);const d=document.createElement('div');d.className='mh-depth-readout';d.textContent='z: 0  ·  scale: 1.00';e.appendChild(d)});
const rsh=()=>{let mb=0;comps.forEach(e=>{mb=Math.max(mb,parseFloat(e.style.top||0)+e.offsetHeight)});stage.style.minHeight=Math.max(mb+120,innerHeight)+'px'};
if(!isDeck)rsh();
if(window.ResizeObserver){let q=false;const ro=new ResizeObserver(()=>{if(q)return;q=true;requestAnimationFrame(()=>{q=false;comps.forEach(snapH);if(!isDeck)rsh()})});comps.forEach(e=>ro.observe(e))}
const isHelp=e=>e.classList&&(e.classList.contains('mh-resize-handle')||e.classList.contains('mh-depth-readout'));
const walk=(root,cb)=>{const w=document.createTreeWalker(root,NodeFilter.SHOW_ELEMENT,{acceptNode(e){if(isHelp(e))return NodeFilter.FILTER_REJECT;for(const n of e.childNodes)if(n.nodeType===3&&n.textContent.trim())return NodeFilter.FILTER_ACCEPT;return NodeFilter.FILTER_SKIP}});let n;while(n=w.nextNode())cb(n)};
const setEd=on=>{comps.forEach(c=>walk(c,e=>e.removeAttribute('contenteditable')));if(!on)return;comps.forEach(c=>{if(isDeck){const s=c.closest('.mh-slide');if(s&&!s.classList.contains('mh-current'))return}walk(c,e=>e.setAttribute('contenteditable','true'))})};
let cur=0,dc=null;
const show=i=>{if(!slides.length)return;cur=((i%slides.length)+slides.length)%slides.length;slides.forEach((s,k)=>s.classList.toggle('mh-current',k===cur));if(dc)dc.textContent=(cur+1)+' / '+slides.length;if(mm)setEd(true)};
if(isDeck&&slides.length){dc=document.createElement('div');dc.className='mh-deck-counter';document.body.appendChild(dc);show(0);requestAnimationFrame(()=>document.body.classList.add('mh-deck-ready'))}
document.addEventListener('keydown',e=>{
  const t=(e.target.tagName||'').toLowerCase();
  if(e.target.isContentEditable&&e.key!=='m'&&e.key!=='M')return;
  if(t==='input'||t==='textarea')return;
  if(e.key==='m'||e.key==='M'){mm=!mm;document.body.classList.toggle('mh-manual-mode',mm);setEd(mm);if(!mm&&document.activeElement&&document.activeElement.blur)document.activeElement.blur();return}
  if(isDeck){
    if(e.key==='ArrowRight'){show(cur+1);e.preventDefault();return}
    if(e.key==='ArrowLeft'){show(cur-1);e.preventDefault();return}
    if(!mm&&(e.key===' '||e.key==='PageDown')){show(cur+1);e.preventDefault();return}
    if(!mm&&e.key==='PageUp'){show(cur-1);e.preventDefault();return}
  }
  if(!mm||!sel)return;
  if(e.key==='ArrowUp'||e.key==='ArrowDown'){const s=getS(sel);s.z+=e.key==='ArrowUp'?Z:-Z;setS(sel,s);e.preventDefault()}
});
document.addEventListener('pointerdown',e=>{
  if(!mm)return;
  const c=e.target.closest('.mh-component');if(!c)return;
  const h=e.target.classList.contains('mh-resize-handle');
  pd={el:c,mode:h?'resize':'drag',sx:e.clientX,sy:e.clientY,st:getS(c),pid:e.pointerId,act:false};
  if(h){pd.act=true;try{c.setPointerCapture(e.pointerId)}catch{}e.preventDefault()}
});
document.addEventListener('pointermove',e=>{
  if(!pd)return;
  const dx=e.clientX-pd.sx,dy=e.clientY-pd.sy;
  if(!pd.act&&Math.hypot(dx,dy)>DT){
    pd.act=true;
    if(sel&&sel!==pd.el)sel.classList.remove('mh-selected');
    sel=pd.el;sel.classList.add('mh-selected');
    try{pd.el.setPointerCapture(pd.pid)}catch{}
    if(document.activeElement&&document.activeElement!==document.body&&document.activeElement.blur)document.activeElement.blur();
    const g=getSelection&&getSelection();if(g&&g.removeAllRanges)g.removeAllRanges();
  }
  if(!pd.act)return;
  const s={...pd.st};
  if(pd.mode==='drag'){s.x=snap(pd.st.x+dx);s.y=snap(pd.st.y+dy)}
  else{const f=1+(dx+dy)/220;let ns=Math.max(0.2,Math.min(6,pd.st.s*f));const iw=pd.el.offsetWidth;if(iw>0){const tw=Math.max(GRID,Math.round(iw*ns/GRID)*GRID);ns=tw/iw}s.s=ns}
  setS(pd.el,s);e.preventDefault();
});
const end=()=>{if(!pd)return;if(!pd.act){if(sel&&sel!==pd.el)sel.classList.remove('mh-selected');sel=pd.el;sel.classList.add('mh-selected')}try{pd.el.releasePointerCapture(pd.pid)}catch{}pd=null};
document.addEventListener('pointerup',end);document.addEventListener('pointercancel',end);
document.addEventListener('click',e=>{
  if(mm){const t=e.target.closest('a, button, input[type="submit"], input[type="button"]');if(t&&t.closest('.mh-component'))e.preventDefault();return}
  if(isDeck&&document.body.classList.contains('mh-deck-ready')){if(e.target.closest('.mh-hint, a, button, input, textarea, [contenteditable]'))return;show(cur+1)}
},true);
});
</script>
```
