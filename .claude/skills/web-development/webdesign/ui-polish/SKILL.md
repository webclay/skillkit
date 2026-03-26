---
name: ui-polish
description: Tailwind CSS implementation techniques for production-quality UI polish. Use when building or refining marketing pages, landing pages, or any Tailwind-based UI that needs to look crafted rather than generated. Covers borders, typography, layout patterns, and visual structure tricks.
---

# UI Polish - Tailwind Implementation Techniques

Concrete Tailwind CSS tricks for turning AI-generated UI into polished, production-quality work. Sourced from Steve Schoger (Tailwind Labs). Apply these automatically when building or refining any Tailwind-based site.

## When to Use This Skill

- Building a marketing page, landing page, or homepage
- Polishing AI-generated UI that looks generic
- User asks to make something look more refined or "less AI"
- Working on any Tailwind CSS project that needs visual detail

---

## Borders & Shadows

### Ring instead of border (the muddy border fix)
Never use a solid color border alongside a shadow - it looks muddy. Instead use a semi-transparent ring:

```html
<!-- Bad: solid border + shadow = muddy -->
<div class="border border-gray-200 shadow-md">

<!-- Good: ring at low opacity bleeds cleanly into the shadow -->
<div class="ring-1 ring-gray-950/10 shadow-md">
```

Use `ring-gray-950/10` (10% opacity) as the default. For inset rings on top of images, use an absolutely positioned overlay with `ring-inset`.

### Inset ring on images
To put a crisp border on top of (not behind) an image:

```html
<div class="relative">
  <img ...>
  <div class="absolute inset-0 rounded-[inherit] ring-1 ring-inset ring-gray-950/10"></div>
</div>
```

---

## Typography

### Variable Inter with display optical sizing
Never use basic Inter. Always load the variable version from rsms.me with display optical sizing and full font features:

```html
<link rel="stylesheet" href="https://rsms.me/inter/inter.css">
```

```css
:root { font-family: 'Inter var', sans-serif; }
@supports (font-variation-settings: normal) {
  :root { font-family: 'Inter var', sans-serif;
          font-feature-settings: 'cv01', 'cv02', 'cv03', 'cv04', 'cv09', 'cv10';
          font-optical-sizing: auto; }
}
```

- Set `font-feature-settings: 'cv05' 1` to get the single-storey `a` (cleaner look)
- Avoid `cv08` - that's the `l` with a tail, which looks odd
- Use `font-weight: 550` for in-between medium (500) and semibold (600)

### Tracking on large headlines
Tighten letter-spacing for any headline above ~24px:

```html
<h1 class="text-5xl font-[550] tracking-tight">
```

For smaller body text, use default or slightly looser tracking.

### ch units for max-width
Use character-based max-width for readable text columns instead of fixed pixel widths:

```html
<!-- Fits ~40 "zero" characters at the current font size -->
<p class="max-w-[40ch]">

<!-- Apply at the same element where font-size is set -->
<p class="text-4xl max-w-[40ch]">
```

Good defaults: `35ch` (tight), `40ch` (comfortable), `45ch` (wide).

### Monospace eyebrow labels
For section labels/eyebrows above headings:

```html
<p class="font-mono text-xs uppercase tracking-wider text-gray-600">
  Features
</p>
```

### Text wrapping
- `text-balance` - balances line lengths, good for headings and CTAs
- `text-pretty` - prevents orphaned last words in paragraphs
- Try both when wrapping looks off - `text-balance` wins most of the time for short copy

---

## Layout Patterns

### Split hero (left headline + right supporting text)
Avoid centered heroes. Instead, split with more weight on the headline:

```html
<div class="grid grid-cols-5 gap-16 items-start">
  <h1 class="col-span-3 text-5xl font-[550] tracking-tight">
    Your headline here
  </h1>
  <div class="col-span-2">
    <p class="text-base text-gray-600">Supporting text here</p>
    <!-- buttons -->
  </div>
</div>
```

Align the top of the supporting text with the top of the headline (use `items-start`).

### Inline section headings (Stripe/Linear style)
Make the section title flow inline into the supporting sentence - same font size, different color:

```html
<p class="text-4xl max-w-[40ch]">
  <span class="font-medium text-gray-950">Unified revenue tracking. </span>
  <span class="font-medium text-gray-500">See every stream in one place, updated in real time.</span>
</p>
```

Use this instead of a separate title + subtitle stacked layout. Requires a slightly longer supporting sentence to fill the width nicely.

### Left-align everything by default
AI defaults to center-aligned. Switch sections, headings, stats, and feature text to left-aligned unless the design specifically needs centering.

### Page container width
Use `max-w-screen-xl` (1280px) or wider as the page container. Default `max-w-7xl` (1280px) is a safe starting point; go to `max-w-screen-2xl` for more breathing room.

---

## Image & Screenshot Techniques

### Screenshot as hero graphic
Use a real screenshot of the app instead of a fake generated UI:
- Capture at 3x for retina sharpness
- Inset 8px from the container edges
- Match the screenshot border-radius to the container radius (concentric radii)
- Crop to the bottom - zero padding at the bottom so the screenshot sits on the container floor
- Remove the bottom corner radius so it bleeds into the edge

### Well-styled image containers
Instead of a plain white card, give image containers a subtle tinted background:

```html
<div class="bg-gray-950/[.03] rounded-2xl p-2">
  <img class="rounded-xl ring-1 ring-gray-950/10" ...>
</div>
```

Start at 2.5% opacity and increase to 5% if more definition is needed. Drop any border on the container itself - the ring on the image handles definition.

### Repositioning cropped images
When you need to position a zoomed/cropped screenshot, ask Claude to build a temporary inline drag tool:

> "Build a temporary inline tool so I can drag the image around in the cropped area. Once I've set the position, I'll copy the values and you can bake them into the code and remove the tool."

---

## Buttons

### Pill-shaped buttons
Use `rounded-full` for a cleaner, more modern button shape on marketing pages.

### Button sizing
- Hero CTA: ~38px tall, 14px text, `px-5`
- Nav button: ~28px tall, `text-sm`, `px-4`
- Use padding to control height, never fixed `h-[px]`

### Button height equalization (ring vs no-ring)
When one button has a ring and one doesn't, the ring adds 2px and misaligns them. Fix with a wrapper span:

```html
<!-- Primary button (no ring) -->
<button class="rounded-full bg-gray-950 text-white px-5 py-2 text-sm shadow-sm">
  Get started
</button>

<!-- Secondary button (with ring) - wrap in p-px span to compensate -->
<span class="inline-flex p-px rounded-full shadow-sm" style="padding: 1px">
  <button class="rounded-full bg-white text-gray-950 px-[calc(1.25rem-1px)] py-[calc(0.5rem-1px)] text-sm ring-1 ring-gray-950/10">
    Watch demo
  </button>
</span>
```

The `p-px` on the span + reduced inner padding keeps both buttons the same rendered height.

---

## Section & Page Structure

### Canvas grid (decorative border treatment)
Add structural borders that run across the full viewport width between sections, with vertical borders hugging the page container. Creates a Stripe/Tailwind-style gridded feel:

- Horizontal dividers: full `100vw` width, `border-t border-gray-950/5`
- Vertical rails: positioned at the edges of the page container
- Containers (hero screenshot, feature cards): 8px gap from the rails on sides and bottom
- Section headings and uncontained content: extra padding from the rails

### Logo clouds
- No title above the logo cloud - just the logos. Context is clear.
- No section background or top/bottom borders
- Logos in `fill-gray-950`, no opacity reduction
- Full width within the container, evenly spaced

### Stats sections
- Left-align stats, fill full container width
- Regular font weight for the numbers (not bold) at `text-5xl`
- `text-gray-600` for labels
- Vertical dividers between each stat

### Testimonials with portrait backgrounds
- Full-bleed portrait photo as card background
- Dark gradient shim from bottom (so white text reads cleanly)
- White quote text, no stars, no avatar circle
- Quote aligned to the bottom of the card
- Name/title in `text-xs` with generous line-height

---

## Quick Reference Checklist

Before calling a page done, verify:

- [ ] No solid-color borders next to shadows - use `ring-gray-950/10` instead
- [ ] Variable Inter loaded from rsms.me with font features enabled
- [ ] Headlines use `tracking-tight` and weight 550
- [ ] Section headings are left-aligned, not centered
- [ ] Max-width on text blocks uses `ch` units, set on the same element as font-size
- [ ] Eyebrow labels use monospace, uppercase, `tracking-wider`, `text-xs text-gray-600`
- [ ] Hero is split layout, not centered
- [ ] Buttons are pill-shaped, sized with padding not fixed height
- [ ] Screenshots are inset 8px, concentric radius, with inset ring overlay
- [ ] Orphaned words fixed with `text-balance` or `text-pretty`
