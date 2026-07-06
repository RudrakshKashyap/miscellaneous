## [CSS dev tools](https://www.youtube.com/watch?v=Qf_5zmxrxzE)


## rem vs em - 1rem is strictly tied to the root of the document (usually the <html> tag), while 1em adapts to the font size of its immediate parent element.


Because the parent is flex flex-col flex children don't stretch along the main axis by default — they size to their content. So the grid only gets as tall as its tallest row needs.
Add flex-1 min-h-0 to the grid div:

<div className='grid min-h-0 flex-1 grid-cols-[minmax(0,1fr)_486px] gap-6'>
flex-1 — claim the remaining main-axis space.
min-h-0 — opt out of the flex item's min-height: auto, otherwise inner overflow-y-auto (e.g. the form pane at page.tsx:282) can blow past parent height instead of scrolling.

CSS gives every flex item a hidden default: min-height: auto, which means "don't shrink smaller than your content."

Result: the body overflows the parent. Your scroll behavior breaks because the body grew the parent instead of staying inside it

min-h-0 overrides min-height: auto and says: "you may shrink as small as needed."


---

<br />
<br />
<br />

- In the CSS Box Model, an element's background fills the content, padding, and border areas. The margin exists completely outside the border and is always transparent, meaning it shows the background of whatever element sits underneath

## Below diagram is when preflight(border-box)

other one is `box-sizing: content-box;` where width and height applied to content.

<div class="border-red-500 h-10 w-10 m-5 p-2 border-2 border-solid">
</div>


```
 ┌───────────────────────────────────────────────┐
 │  margin  (m-5 = 20px, transparent)            │
 │   ┌─────────────────────────────────────────┐ │
 │   │██ border (border-2 = 2px, red) ████████ │ │
 │   │██  ┌───────────────────────────────┐ ██ │ │
 │   │██  │ padding (p-2 = 8px)           │ ██ │ │
 │   │██  │   ┌───────────────────────┐   │ ██ │ │
 │   │██  │   │                       │   │ ██ │ │
 │   │██  │   │   content 20 × 20     │   │ ██ │ │
 │   │██  │   │                       │   │ ██ │ │
 │   │██  │   └───────────────────────┘   │ ██ │ │
 │   │██  │                               │ ██ │ │
 │   │██  └───────────────────────────────┘ ██ │ │
 │   │████████████████████████████████████████ │ │
 │   └─────────────────────────────────────────┘ │
 │                                               │
 └───────────────────────────────────────────────┘

 ◄──────────────────── 80px footprint ────────────────────►
       ◄──────────── 40px box (w-10) ────────────►
 20px      2px    8px      20px      8px    2px      20px
margin   border  padding  content  padding border   margin
```

**Border-box** means `w-10`/`h-10` measure the **outer edge** (border included), so border + padding eat into the content, shrinking it from 40 → 20. Margin always sits **outside** the box and adds to the footprint.


`<div>` vs `<p>` tag

- `<div>` — generic, **semantic-less** block container. Used for layout/grouping. Can nest anything (including other divs, p, headings, etc.).
- `<p>` — **semantic** paragraph of text. Cannot contain block-level elements — only inline content (text, `<span>`, `<a>`, `<strong>`…).

> Note: the browser's UA stylesheet gives `<p>` a default `margin-block: 1em` (~16px top/bottom), but CSS resets remove it. **Tailwind's Preflight** sets `margin: 0` on every element, so in a Tailwind (or Normalize/reset-based) project you won't see any default `<p>` margin — only in plain unreset HTML.

Key trap: putting a `<div>` (or any block element) inside a `<p>` is invalid HTML — the browser auto-closes the `<p>` before the div, breaking your intended nesting.

> User Agent style sheet also give margin -> 8px top & bottom by default, and we need to disable that.

Use `<p>` for actual prose (accessibility + SEO read it as a paragraph); use `<div>` for structure and styling.


## Difference bw all `display` keyword properties

`display` controls **two things**: how the box behaves in the *outer* flow (block vs inline), and how its *children* are laid out (the inner formatting context).

### The core values

| value | flow | starts new line? | respects w/h? | margin/padding | typical use |
|---|---|---|---|---|---|
| `block` | block | yes (full width) | ✅ all | ✅ all 4 sides | `<div>`, `<p>`, `<section>` |
| `inline` | inline | no (flows in text) | ❌ ignored | ↔ horizontal only* | `<span>`, `<a>`, `<strong>` |
| `inline-block` | inline | no | ✅ all | ✅ all 4 sides | chips, buttons in a row |
| `flex` | block | yes | ✅ | ✅ | 1-D layout (row/col) |
| `inline-flex` | inline | no | ✅ | ✅ | flex box sitting in text |
| `grid` | block | yes | ✅ | ✅ | 2-D layout (rows + cols) |
| `inline-grid` | inline | no | ✅ | ✅ | grid sitting in text |
| `none` | — | removed entirely | — | — | hide (no space reserved) |
| `contents` | — | box disappears, children promoted | — | — | unwrap a wrapper |
| `table` / `table-*` | block | yes | ✅ | ✅ | mimic `<table>` behavior |
| `list-item` | block | yes | ✅ | ✅ | generates a marker (•) |
| `flow-root` | block | yes | ✅ | ✅ | new BFC → clears floats |

\* `inline`: vertical margin/padding **render visually** but don't push surrounding lines apart, and width/height are simply ignored.

### What each value actually does

**`block`** — the box becomes a vertical brick.
- Takes the **full width** of its parent (even if content is tiny).
- Forces a line break before and after → siblings stack top-to-bottom.
- Honors `width`, `height`, and all four margins/paddings.
```
[ block A ──────────────────────────── ] (full width)
[ block B ──────────────────────────── ]
```

**`inline`** — the box flows inside a line of text, like a word.
- **No line break** — sits next to other inline content.
- **Ignores `width`/`height`** and vertical margins (you can't size it).
- Only horizontal margin/padding push neighbors.
```
text text [inlineA][inlineB] text text wraps →
```

**`inline-block`** — flows on a line *but* is a real box.
- Sits next to neighbors (no forced line break) like `inline`…
- …**but respects `width`/`height` and all margins/padding** like `block`.
- Use for sized things in a row (badges, nav items) without flex/grid.
```
[ boxA ] [ boxB ] [ boxC ]   ← all on one line, each sized
```

**[`flex`](https://www.youtube.com/watch?v=wsTv9y931o8)** — turns the element into a **1-D layout container**.
- The element itself is still block-level.
- Its **direct children become flex items**, laid out along one axis (`flex-direction: row` default, or `column`).
- Unlocks `justify-content`, `align-items`, `gap`, `flex-grow/shrink`.
```
flex-direction: row →  [child][child][child]
flex-direction: col ↓  [child]
                       [child]
```

**`inline-flex`** — same flex behavior for the **children**, but the container itself flows **inline** (sits inside a line of text instead of taking full width).

#### Properties that go on flex **items** (the children, not the container)

> Container props (`justify-content`, `align-items`(default strech), `gap`, `flex-direction`) live on the parent. These below live on each **child**.

- **`flex-grow`** (default `0`) — how much of the **leftover free space** this item soaks up, as a ratio vs siblings. `flex-grow: 1` on all = equal share; `2` takes twice as much as a `1`.
- **`flex-shrink`** (default `1`) — how much the item is allowed to **shrink** when there isn't enough room. `0` = never shrink (can cause overflow); higher = gives up size faster.
- **`flex-basis`** (default `auto`) — the item's **starting/ideal size** along the main axis *before* `grow`/`shrink` redistribute the leftover space. Think of it as "what size do I *want* to be?"; grow/shrink then adjust from there.
  - `auto` → fall back to the item's `width` (or `height` in a column), or its content size if neither is set.
  - a length/percent (`200px`, `30%`) → that becomes the base size, and it **overrides `width`** on the main axis when both are set.
  - `0` → ignore content entirely; the item's size is decided **purely by `flex-grow` ratios** (this is why `flex: 1` = `1 1 0%` gives perfectly equal columns).
  - `content` → size to the content (less common).

  **The 3-step flex sizing algorithm:**
  1. Lay every item out at its `flex-basis`.
  2. Add up the bases. Compare to the container's main size.
  3. **Leftover space?** → distribute by `flex-grow`. **Not enough room?** → remove by `flex-shrink`.

  ```
  container = 600px, gap 0, three items
  basis 0,    all grow:1  → [ 200 ][ 200 ][ 200 ]   (content ignored, pure equal split)
  basis auto, all grow:0  → [80][120][60]  …240px used, 360px empty (sized to content)
  basis 100px,all grow:1  → start 100·3=300, 300 left → +100 each → [200][200][200]
  basis 200px,all shrink:1→ want 600 in a 450 box → over by 150 → −50 each → [150][150][150]
  ```

> Setting width: 300px locks the element's baseline to that size. Conversely, setting flex-basis: 300px means: "Start at 300px, then modify my size if flex-grow or flex-shrink dictates it.

  **`width` vs `flex-basis`:** on the main axis `flex-basis` wins. `width` only matters when `flex-basis: auto`. Set `flex-basis: 0` and you neutralize `width` for grow-based sizing.
- **`flex`** — shorthand for the three above. Memorize these:
  - `flex: 1` → `1 1 0%` — all items equal width, ignore content size (classic "fill equally").
  - `flex: auto` → `1 1 auto` — grow/shrink but **start from content size** (bigger content → bigger item).
  - `flex: none` → `0 0 auto` — fixed, never grow or shrink (rigid).
  - `flex: 0 1 auto` → the **default** (shrink only).
- **`align-self`** — override the container's `align-items` for **this one item** on the cross axis (`auto | flex-start | center | flex-end | stretch | baseline`). (no property `justify-self` for flex-item, it is avaiable in grid though)
- **`order`** (default `0`) — reorder items visually without touching the DOM; lower renders first, negatives allowed. ⚠️ visual only — screen readers & tab order still follow DOM.

```
container: flex, width 300, gap 0
items all flex:1   → [ 100 ][ 100 ][ 100 ]   (equal share of space)
middle flex:2      → [ 75 ][  150  ][ 75 ]    (middle takes 2× the free space)
middle flex:none(80)→[ 110][  80  ][ 110 ]    (middle rigid, others split rest)
```

**Main axis vs cross axis** (depends on `flex-direction`):
- `row` → main = horizontal, cross = vertical. `grow/shrink/basis` act on **width**; `align-self` on **height**.
- `column` → main = vertical, cross = horizontal. They swap.

**`flex-wrap` & `align-content`** — by default flex is single-line (`flex-wrap: nowrap`): items shrink/overflow rather than wrap. With **`flex-wrap: wrap`**, items that don't fit move to a **new line**, so you now have *multiple lines* stacked along the cross axis.

Mental shortcut: **`align-content` is to the cross axis what `justify-content` is to the main axis** — both distribute *free space between things*. `justify-content` spaces **items along the main axis**; `align-content` spaces **lines along the cross axis**.

| axis | spacing items in a line | spacing the lines themselves |
|---|---|---|
| main | `justify-content` | — |
| cross | `align-items` (per line) | `align-content` (the whole stack) |

- **`justify-content`** — distributes items along the **main axis** (the direction they flow).
- **`align-items`** — aligns items *within* a single line on the **cross axis**.
- **`align-content`** — distributes the **whole group of lines** along the **cross axis** (`flex-start | center | space-between | space-around | stretch`…). **Only has an effect when there are 2+ lines** (wrapping on); ignored on a single line.

```
flex-wrap: wrap, justify-content: end, 5 items, narrow container
            [A][B][C]   ← line 1   ┐
               [D][E]   ← line 2   ┘ align-content spaces THESE lines (cross axis)
 └─ justify-content: end pushes items to the END of the main axis (right)
    align-items aligns items WITHIN each line (cross axis)
```

**[`grid`](https://www.youtube.com/watch?v=JYfiaSKeYhE)** — turns the element into a **2-D layout container** (rows *and* columns at once).
- Children placed into a defined grid via `grid-template-columns/rows`.
- Best when you control both axes (page layouts, card galleries).
```
grid-template-columns: 1fr 1fr 1fr
┌──────┬──────┬──────┐
│  A   │  B   │  C   │
├──────┼──────┼──────┤
│  D   │  E   │  F   │
└──────┴──────┴──────┘
```

## [Grid vs Flex](https://youtu.be/aEj6k-gi9-s?si=hbTvCB2bssY21oWs&t=227)

<br/>

**`inline-grid`** — grid for the children, but the container flows inline.

**`none`** — element (and all descendants) **completely removed** from layout and the accessibility tree. No space reserved. Common for toggling visibility / responsive hiding.

**`contents`** — the element's **own box disappears**, but its children render as if they were direct children of the grandparent. Lets a wrapper "vanish" so its kids participate directly in a parent flex/grid. ⚠️ has had a11y bugs.

**`flow-root`** — block box that establishes a fresh **Block Formatting Context**: it fully **contains floats** (modern clearfix) and stops margins from collapsing through it.

**`table` / `table-row` / `table-cell` …** — make non-`<table>` elements behave like table parts (equal-height columns, cell alignment) without using table markup.

**`list-item`** — renders like a block but also generates a **list marker** (the `•` or number); what `<li>` uses by default.

### Key mental model

- **Outer display** (`block` vs `inline`) = how the box sits among siblings.
- **Inner display** (`flow`, `flex`, `grid`, `table`) = how its kids are arranged.
  CSS3 actually defines `display: block flex` (two-value), and the old keywords are shorthands: `flex` = `block flex`, `inline-flex` = `inline flex`.

### The three you'll confuse

- **`block`** — greedy: takes full available width, stacks vertically, honors every box-model property.
- **`inline`** — flows like words; **w/h ignored**, only horizontal margin/padding affect layout. Can't be given a fixed size.
- **`inline-block`** — best of both: flows inline (sits next to other content) **but** respects width/height and all margins/padding. Use when you want sized boxes on one line without flex/grid.

### Special / often-forgotten

- **`none`** — element + children removed from render tree and accessibility tree; reserves **no space** (vs `visibility:hidden` which keeps the space, and `opacity:0` which keeps space + stays interactive).
- **`contents`** — the element's own box vanishes but its children remain, as if they were direct children of the grandparent. Handy to make a wrapper "transparent" to a flex/grid parent. ⚠️ historically buggy for accessibility.
- **`flow-root`** — establishes a new **Block Formatting Context**: contains floats (modern `clearfix`) and prevents margin-collapse with children.
- **`flex` / `grid`** — change only how *children* lay out; the element itself is still a block by default.


### `em` is relative to its parent element, while `rem` is relative to the root (`<html>`) element.
### (%) represents a relative measurement unit that calculates its size based on a `parent` element's property value, height should be expliclity defined in parent for it to work

# zoom in - out (Ctrl +/−)

**TL;DR:** page zoom scales the **CSS reference pixel itself**, so *everything* grows together — `px`, `em`, `rem`, `%`, images, layout. It does **not** change any `font-size` value or computed style.

**What it changes**
- **Everything scales proportionally**, including absolute `px`. At 150% zoom a `16px` font *renders* like `24px`, a `100px` box like `150px`.
- **`em` / `rem` scale too** — not because the root font-size changed, but because the px each resolves to is drawn bigger. `1rem` is still "16px" in CSS terms.
- **Media queries can fire** — zoom changes the *effective* viewport width in CSS px, so zooming in ≈ shrinking the viewport. `@media (max-width: 768px)` can trigger.

**What it does NOT change**
- No `font-size`, root size, or computed length changes. `getComputedStyle` still reports `16px`. Zoom is a render-layer multiplier, not a style change.

**Key contrast — page zoom vs. browser's default/minimum font-size setting**
| | scales `px`? | changes what `1rem` = ? | affects `rem`/`em` only? |
|---|---|---|---|
| **Page zoom (Ctrl +/−)** | ✅ yes | ❌ no | no — scales everything |
| **Default font-size setting** (a11y) | ❌ no | ✅ yes (e.g. 16→20px) | ✅ yes — `px` stays fixed |

So the advice *"use `rem` for font sizes so users get larger text"* is about the **font-size setting**, not Ctrl-zoom. Ctrl-zoom enlarges a `px`-based site fine; `rem` matters for users who raise their browser's **default text size** for accessibility.

### Does changing window size change styles given by vh / vw?

**Yes — they recalculate live as the viewport resizes.** `vw`/`vh` are relative to the *viewport*, not the page or any parent:
- `1vw` = 1% of viewport **width**, `1vh` = 1% of viewport **height**. So `50vw` is always half the window width — resize the window and it instantly re-measures.
- Unlike `%` (relative to the **parent** element), `vw/vh` ignore the parent entirely and track the window.

**Contrast with zoom:** resizing the window changes the *actual* viewport size → `vw/vh` change. Ctrl-zoom changes the *CSS px* but the viewport stays the same number of CSS px, so `vw/vh` values stay constant (they just render bigger like everything else).

# **Mobile gotcha:** `100vh` includes the area under the browser's address bar on phones, causing the "jumpy 100vh" problem. Newer units fix it: **`svh`** (small — bar visible), **`lvh`** (large — bar hidden), **`dvh`** (dynamic — adjusts as the bar shows/hides).

<br/>
<br/>
<br/>

## [position](https://youtu.be/YEmdHbQBCSQ?si=cGV4Z0QSENdhou84): relative / absolute (and static, fixed, sticky) - [grid stacking](https://youtu.be/JYfiaSKeYhE?si=Jiycnit_z7-NM0ja&t=1072) is cool

`position` controls how an element is placed and what `top/right/bottom/left` (the **insets**) are measured against.

| value | in normal flow? | insets relative to | leaves a gap? |
|---|---|---|---|
| `static` (default) | yes | n/a — insets ignored | — |
| `relative` | yes | **its own original spot** | yes (keeps its space) |
| `absolute` | **no** (removed) | nearest **positioned** ancestor | no (collapses) |
| `fixed` | no (removed) | the **viewport** | no |
| `sticky` | yes | scroll container (toggles relative↔fixed) | yes |

**`relative`** — stays in flow, occupies its original space, then insets *nudge* it visually from that origin (other elements don't move). Its main job in practice: become the **positioning context** (anchor) for an `absolute` child.

**`absolute`** — yanked out of flow (siblings close the gap), positioned against the **nearest ancestor with `position` other than `static`** — i.e. the nearest `relative/absolute/fixed/sticky` parent. If none exists, it falls back to the initial containing block (≈ the page/`<html>`).

```
.parent { position: relative; }      ← the anchor
  .child { position: absolute; top:0; right:0; }   ← pinned to parent's top-right

┌─ parent (relative) ───────────┐
│                        [child] │  ← child sits in corner, out of flow
│  normal content flows here     │
└────────────────────────────────┘
```

**The classic pattern:** parent `relative` + child `absolute` = pin the child anywhere inside the parent (badges, close buttons, dropdowns, overlays).

- **`fixed`** — like absolute but anchored to the **viewport**; stays put on scroll (sticky headers, modals).
- **`sticky`** — hybrid: behaves `relative` until you scroll past a threshold (e.g. `top: 0`), then "sticks" like `fixed` within its scroll container.


## `<aside>` vs `<div>` — is it just a renamed div?

Visually/layout-wise: **yes, identical** — `<aside>` is a block element with the same default box behavior as `<div>`, no special styling.

The difference is **semantics** (meaning for browsers, screen readers, SEO):

- **`<div>`** — zero meaning, just a generic container.
- **`<aside>`** — "content tangential to the main content" (sidebars, related links, callouts, side menus). It carries an implicit ARIA role of **`complementary`**, a landmark — screen-reader users can jump to it via landmark navigation, and assistive tech announces it as a distinct region.

So swapping `<aside>` for `<div>` changes nothing on screen but loses that accessibility landmark. Use `<aside>` (and `<nav>`, `<header>`, `<main>`, `<footer>`) when the region has a recognized role; use `<div>` for pure layout/styling wrappers.

> Note: a page can have multiple `<aside>`s, but if so give each an `aria-label`/`aria-labelledby` so the landmarks are distinguishable.