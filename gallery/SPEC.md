# SPEC — El Araby plate-screening admin overview ("النظرة العامة")

You are building ONE design out of ten. Every design shares this contract.
What differs between designs is the **organizing idea** — the geometry and information
architecture of the screen. Not colours. Not fonts. Not button placement.

The user has rejected twelve previous designs with the words *"انت واخد نفس التصميم و نفس
اماكن ال button"* — same skeleton, reskinned. **Your design must not be a sidebar + KPI
cards + table.** If your layout could be described as "left nav, four stat cards across the
top, a chart, and a table underneath", you have failed regardless of how it looks.

---

## 0. Output

Write exactly one self-contained file to the absolute path you are given.
No build step. No imports except the Google Fonts stylesheet below. No frameworks.
Vanilla HTML + CSS + JS in one file. It must open correctly from `file://`.

Required in `<head>`, verbatim:

```html
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans+Arabic:wght@300;400;500;600;700&display=swap" rel="stylesheet">
```

`<html lang="ar" dir="rtl">`. Title = the Arabic name of your design.

---

## 1. Material — non-negotiable

The user uploaded a reference image and said: **"glassmorphism only, with no colour except in
the background."** That is the law.

**The scene** (behind everything) is dark green and carries all the colour:

```css
body { background: #06100c; }
```

Over that, place 4–6 large soft light sources — radial gradients, `filter: blur(90px)`,
opacity between `0.10` and `0.22`. They are the only saturated colour on the page. Use
greens with slight hue drift so the scene has depth rather than one flat wash:

```
#0f4d38  #10613f  #0b3a30  #145a45  #08302a
```

Add a vignette (a large inset radial that darkens the edges) and a fine grain overlay
(SVG `feTurbulence`, `opacity: .035`, `pointer-events: none`, fixed).

**Keep the lighting calm.** The user asked twice for `اضاءة اهدى`. No hotspot should read as
a bright blob. If in doubt, lower the opacity.

**Every panel is neutral glass and has no hue of its own:**

```css
background: rgba(255,255,255,.045);
backdrop-filter: blur(22px) saturate(1.25);
-webkit-backdrop-filter: blur(22px) saturate(1.25);
border: 1px solid rgba(255,255,255,.085);
border-radius: 16px;
box-shadow: 0 1px 0 0 rgba(255,255,255,.10) inset,   /* top sheen */
            0 20px 50px -20px rgba(0,0,0,.55);
```

Vary blur (14px → 34px) and alpha (.03 → .07) between panels to build depth — nearer panels
sharper and lighter, farther panels blurrier and dimmer. Never give a panel a green tint;
the green must come from the scene showing *through* it.

**Glow** is allowed and wanted, but tinted by the palette, never neon:
`box-shadow: 0 0 40px -12px rgba(110,196,155,.35)` on the one element that deserves emphasis.
No more than two glowing elements on screen.

---

## 2. Tokens

```css
:root{
  --ink:      #f1f6f3;   /* primary text  */
  --ink-2:    #bac6c0;   /* secondary     */
  --ink-3:    #87938c;   /* tertiary/meta */
  --line:     rgba(255,255,255,.09);
  --line-2:   rgba(255,255,255,.05);

  --accent:   #6ec49b;   /* the only chromatic UI colour, used sparingly */

  --wanted:   #ff6b6b;   /* مطلوب  — product law: red      */
  --clean:    #3ddc97;   /* سليم   — product law: green    */
  --unread:   #e8b055;   /* غير مقروء — amber              */

  /* validated colourblind-safe series set (ΔE ≥ 8 under protan/deutan) */
  --s1: #199e70;  --s2: #3987e5;  --s3: #d55181;  --s4: #9085e9;
}
```

`--clean` is deliberately brighter and more saturated than the scene green so "سليم" still
reads as a *status* and not as the background. Do not swap these two.

`--wanted` red and `--clean` green are product law and may not be reassigned.

---

## 3. Type

`font-family: 'IBM Plex Sans Arabic', system-ui, sans-serif;`

- Figures: `font-variant-numeric: tabular-nums; font-feature-settings:"tnum" 1;`
- **Western digits everywhere** (`24`, not `٢٤`). Consistent across all ten designs.
- Numeric axis labels inside SVG need `direction: ltr; unicode-bidi: isolate;` or bidi will
  split them ("00" renders as "0 0").
- Line-height for Arabic body text ≥ 1.7 — Arabic needs more leading than Latin.
- Letter-spacing on Arabic must stay `normal` (it breaks the joins).

---

## 4. Animation — exactly three kinds, no more

The user chose these three and explicitly ruled out continuous background motion:

1. **Page entrance** — panels rise and fade in, staggered 40–70ms apart,
   `cubic-bezier(.22,.61,.36,1)`, 420–700ms. Runs once.
2. **Numbers and charts** — counters count up; lines draw via `stroke-dasharray`; bars/arcs
   grow from their baseline. Runs once on load and again when data changes.
3. **Interaction** — hover, press, focus, selection. 120–200ms.

**Forbidden:** anything with `animation-iteration-count: infinite` that is decorative —
drifting orbs, pulsing glows, shimmering borders, breathing gradients, a looping radar sweep.
A single small "live" dot may pulse; that is the only exception.

Every animation must be disabled under:

```css
@media (prefers-reduced-motion: reduce){ *,*::before,*::after{
  animation-duration:.01ms !important; animation-iteration-count:1 !important;
  transition-duration:.01ms !important; } }
```

Real figures must be present in the HTML markup. Animate *on top of* them — never start a
counter from an empty element, because `requestAnimationFrame` is throttled in a background
tab and the number would be stuck at 0.

---

## 5. Density and required content

The user said the earlier work was **too empty**. Fill the screen with real information.
Every design must surface all of the following, arranged however your concept dictates:

- the six headline figures
- the 24-hour series (see §7)
- all nine active gates with their volumes
- at least twelve recent events
- the operator/reviewer dimension
- gate health (2 of 11 gates are down — this must be visible somewhere)

## 6. Interaction — at least two that change real state

Not tooltips-only. Clicking something must visibly re-filter, re-scope, or re-render
something else. Keep state in JS and re-render from it. Examples: selecting a gate filters
the event list and re-draws the series; a verdict chip filters; a time-range control rescopes
everything; clicking an hour drills into that hour.

Provide an honest empty state for any filter that yields nothing:
`لا توجد سجلات مطابقة`.

Keyboard: every interactive element must be a real `<button>`/`<a>` or carry `tabindex="0"`
plus Enter/Space handling, and must show a visible `:focus-visible` ring.

---

## 7. The dataset — copy this verbatim

All ten designs use identical numbers so they can be compared fairly.

```js
const ADMIN = { name: "أحمد عماد", role: "مدير النظام", initial: "أ" };
const UPDATED = "اليوم 14:32";

const KPI = {
  total:   { v: 12847, label: "إجمالي عمليات المسح", delta: +6.2 },
  wanted:  { v: 38,    label: "مركبات مطلوبة",        delta: -11.6 },
  clean:   { v: 12782, label: "مركبات سليمة",         delta: +6.3 },
  unread:  { v: 27,    label: "لوحات غير مقروءة",     delta: +2.4 },
  latency: { v: 412,   label: "متوسط زمن المعالجة", unit: "مللي/ث", delta: -8.0 },
  gates:   { v: 9,     of: 11, label: "بوابات نشطة" }
};

// index = hour 0..23
const HOURLY = {
  total:  [218,164,131,119,147,268,512,753,874,846,752,724,703,658,691,709,802,858,804,651,487,392,314,270],
  wanted: [1,0,1,0,0,1,2,3,4,2,2,1,2,1,2,2,3,4,3,2,1,0,1,0],
  unread: [0,1,0,1,0,1,1,2,2,2,1,1,2,1,1,1,2,2,2,1,1,1,0,1]
};

const GATES = [
  { id:"g1", name:"بوابة الشمال",     total:2413, wanted:8, unread:4, ms:388, status:"online"   },
  { id:"g2", name:"الميناء 3",        total:2148, wanted:9, unread:6, ms:441, status:"online"   },
  { id:"g3", name:"طريق الملك فهد",   total:1876, wanted:5, unread:3, ms:402, status:"online"   },
  { id:"g4", name:"بوابة الجنوب",     total:1542, wanted:4, unread:2, ms:371, status:"online"   },
  { id:"g5", name:"مدخل المستودع",    total:1288, wanted:3, unread:4, ms:456, status:"online"   },
  { id:"g6", name:"بوابة الشرق",      total:1104, wanted:3, unread:2, ms:419, status:"online"   },
  { id:"g7", name:"محطة الوزن",       total: 936, wanted:2, unread:3, ms:508, status:"degraded" },
  { id:"g8", name:"ساحة الحجز",       total: 802, wanted:3, unread:2, ms:394, status:"online"   },
  { id:"g9", name:"بوابة المطار",     total: 738, wanted:1, unread:1, ms:366, status:"online"   },
  { id:"g10",name:"بوابة الغرب",      total:   0, wanted:0, unread:0, ms:  0, status:"offline"  },
  { id:"g11",name:"مدخل الصيانة",     total:   0, wanted:0, unread:0, ms:  0, status:"maint"    }
];
// status labels: online "تعمل" · degraded "بطيئة" · offline "خارج الخدمة" · maint "صيانة"

const OPERATORS = [
  { name:"خالد المطيري",  reviews:412, wanted:11, acc:96 },
  { name:"نورة السبيعي",  reviews:388, wanted: 9, acc:98 },
  { name:"فهد القحطاني",  reviews:351, wanted: 7, acc:94 },
  { name:"ريم الدوسري",   reviews:306, wanted: 5, acc:97 },
  { name:"سعود العتيبي",  reviews:274, wanted: 4, acc:92 },
  { name:"لطيفة الحربي",  reviews:218, wanted: 2, acc:95 }
];

// verdict: "wanted" مطلوب · "clean" سليم · "unread" غير مقروء
const EVENTS = [
  { t:"14:31", plate:"ر ن ب 4582", gate:"g2", verdict:"wanted", reason:"بلاغ سرقة",            ref:"KSA-2291", by:"نورة السبيعي" },
  { t:"14:29", plate:"أ ل م 7123", gate:"g1", verdict:"clean" },
  { t:"14:27", plate:"ك ط د 9046", gate:"g3", verdict:"clean" },
  { t:"14:26", plate:"ص ع ه 1187", gate:"g7", verdict:"unread", reason:"إضاءة منخفضة" },
  { t:"14:24", plate:"ب ي و 6350", gate:"g4", verdict:"clean" },
  { t:"14:22", plate:"د ر ق 2708", gate:"g8", verdict:"wanted", reason:"مخالفات متراكمة",      ref:"KSA-2287", by:"فهد القحطاني" },
  { t:"14:20", plate:"ه م ن 5419", gate:"g5", verdict:"clean" },
  { t:"14:18", plate:"ط س ل 8264", gate:"g6", verdict:"clean" },
  { t:"14:17", plate:"و ح أ 3095", gate:"g2", verdict:"clean" },
  { t:"14:15", plate:"ق ب ع 7731", gate:"g1", verdict:"wanted", reason:"قائمة مراقبة داخلية", ref:"KSA-2284", by:"ريم الدوسري" },
  { t:"14:13", plate:"ل د ص 4426", gate:"g9", verdict:"clean" },
  { t:"14:11", plate:"ن أ ط 6802", gate:"g3", verdict:"unread", reason:"لوحة متسخة" },
  { t:"14:09", plate:"ع ك ر 1573", gate:"g4", verdict:"clean" },
  { t:"14:07", plate:"م و ه 9218", gate:"g2", verdict:"clean" },
  { t:"14:05", plate:"س ق ب 2947", gate:"g5", verdict:"wanted", reason:"بلاغ اختطاف",          ref:"KSA-2280", by:"سعود العتيبي" },
  { t:"14:03", plate:"ح ل ي 5064", gate:"g6", verdict:"clean" }
];
```

Number formatting: `n.toLocaleString('en-US')` → `12,847`.

---

## 8. RTL correctness — these have all bitten us before

- Use **logical properties**: `margin-inline-start`, `padding-inline-end`,
  `border-start-end-radius`, `inset-inline-start`. Never `left`/`right`/`margin-left`.
- **An SVG `viewBox` is always LTR.** `dir="rtl"` does not mirror SVG internals. If you want a
  chart to read right-to-left, reverse the data or apply `transform="scale(-1,1)"` to the plot
  group and un-mirror the text. Pointer maths inside SVG is always
  `e.clientX - rect.left` — never `rect.right - e.clientX`.
- Wrap any Latin/numeric run inside Arabic text in
  `<span style="unicode-bidi:isolate">` or it will jump position.
- Plates mix Arabic letters and Western digits — render each plate inside an element with
  `unicode-bidi: isolate; direction: rtl;` so the digits stay put.
- Mixed-script labels like `412 مللي/ث` need isolation on the number.

---

## 9. Craft floor — a reviewer will check these

- Contrast: body text ≥ 4.5:1 against the glass-over-scene composite; large text ≥ 3:1.
  Glass at `.045` alpha over `#06100c` composites to roughly `#151d19` — check against that,
  not against white.
- No dual-axis charts. Ever. If two series have different magnitudes (total ~800 vs wanted
  ~4), use two stacked charts sharing one x-axis. Scaling one series to fit the other is the
  same sin wearing a hat.
- Axis maxima must be "nice" numbers (1 / 2 / 2.5 / 5 / 10 × 10ⁿ), not `ceil(max)`.
- Bars ≤ 24px thick, lines 2px, 2px gaps between adjacent marks.
- No decorative icon fonts or emoji as UI. Inline SVG only, `stroke-width: 1.5`,
  `currentColor`.
- No `title` attributes standing in for real tooltips.
- Scrollbars styled or hidden consistently; never a horizontal body scrollbar.
- Everything works at 1440×900 and degrades sanely down to 390px wide.

---

## 10. Footer — required on every design

A single quiet line, `--ink-3`, at the very bottom:

```
بيانات توضيحية — ليست بيانات حقيقية · نظام العربي لفحص اللوحات
```

Do not put a real logo, a real client name beyond "العربي", or anything that could be
mistaken for a live production screen.
