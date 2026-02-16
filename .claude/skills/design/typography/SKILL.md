---
name: typography
description: Web typography rules and best practices based on Butterick's Practical Typography. Trigger words - typography, font size, line height, line spacing, line length, font pairing, heading size, body text, letter spacing, kerning, paragraph spacing, readable, readability, typographic scale, text styling
---

# Typography

Actionable typography rules for web projects. Based on Butterick's Practical Typography. Apply these rules when building or reviewing UI - they work with any CSS framework (Tailwind, vanilla CSS, CSS-in-JS).

## When to Use This Skill

- Setting up body text, headings, or typographic scale
- Reviewing or improving readability of a page
- Choosing font sizes, line heights, or line lengths
- Pairing fonts or choosing typefaces
- Styling text elements (lists, blockquotes, tables, caps)
- User mentions typography, readability, or text styling

## The Five Fundamentals

Every typography decision starts with these five properties for body text. Get these right and the page is 80% there.

### 1. Font Size (point size)

**Web body text: 15-25px.** Different fonts render differently at identical sizes - adjust visually, don't just trust the number.

```css
/* Good starting points */
body { font-size: 18px; }  /* Most fonts */
body { font-size: 16px; }  /* Larger x-height fonts like Inter */
body { font-size: 20px; }  /* Smaller fonts like Garamond */
```

**Tailwind:** `text-base` (16px) to `text-lg` (18px) for body. Adjust with `text-[17px]` if needed.

**Headings:** Use the smallest increment necessary. If body is 18px, try 20-22px for H3, not 28px. Avoid the web default of headings at 200% of body.

### 2. Line Height (line spacing)

**120-145% of the font size.** This is the single biggest readability lever.

```css
/* Unitless values (recommended) */
body { line-height: 1.4; }   /* 140% - good default for body */
h1   { line-height: 1.2; }   /* Tighter for headings */
h2   { line-height: 1.25; }
```

**Tailwind:** `leading-normal` (1.5) is too loose for most fonts. Use `leading-relaxed` (1.625) only for small text. Prefer `leading-snug` (1.375) or `leading-[1.4]`.

**Rules of thumb:**
- Longer lines need more line height
- Larger font sizes need less line height (percentage-wise)
- Dark backgrounds need slightly more line height

### 3. Line Length

**45-90 characters per line (including spaces).** This is the optimal reading width. The #1 mistake in web typography is text that stretches edge-to-edge.

```css
/* Constrain text width */
.prose { max-width: 65ch; }  /* ~65 characters - good default */
p      { max-width: 75ch; }  /* Slightly wider for larger fonts */
```

**Tailwind:** `max-w-prose` (65ch) is already correct. Use it.

**Quick test:** Type the alphabet 2-3 times on a line. If fewer than 2 alphabets fit, lines are too short. If more than 3 fit, lines are too long.

**Responsive:** Maintain 45-90 characters at every breakpoint. Use `max-width` to prevent stretching on wide screens. On mobile, inherent screen width usually handles the minimum.

### 4. Font Choice

**Serif or sans-serif both work on the web.** Modern screens render both well. The key is to use a professional typeface, not which category.

**Quality free fonts for web:**
- **Sans-serif:** Inter, IBM Plex Sans, Source Sans, DM Sans
- **Serif:** Source Serif, IBM Plex Serif, Charter, Lora
- **Monospace:** JetBrains Mono, IBM Plex Mono, Source Code Pro, Fira Code

**Avoid:** Comic Sans, Papyrus, Arial (use Helvetica or a proper alternative), default Times New Roman, Verdana at large sizes.

**System font stack (acceptable fallback):**
```css
font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```

### 5. Text Color

**Don't use pure black on screens.** Screens emit light, creating harsh contrast with `#000`. Use dark gray for body text.

```css
body { color: #333; }        /* Safe default */
body { color: #374151; }     /* Tailwind gray-700 - good */
body { color: #1f2937; }     /* Tailwind gray-800 - slightly darker */
```

**Tailwind:** `text-gray-700` or `text-gray-800` for body text. Reserve `text-black` for headings if needed.

**Color for links only:** Reserve color primarily for clickable elements. Don't use color on non-clickable text - it confuses readers.

## Headings

### Sizing

- **Use the smallest size increase that creates clear hierarchy.** If body is 18px, H3 at 20px works. You don't need 36px H1s.
- **Limit to 2-3 heading levels.** If you need more, your content structure needs work, not more heading sizes.
- **Spacing above headings matters more than size.** A heading with generous `margin-top` stands out even without dramatic size increases.

### Styling

- **Bold, not italic** for headings. Sans-serif italics barely look different from roman.
- **No underline** on headings.
- **No all-caps** for headings longer than one line (creates a wall of rectangles).
- **Suppress hyphenation** in headings: `hyphens: none`.
- **Keep headings with their content:** `break-after: avoid` or `page-break-after: avoid`.

```css
h1, h2, h3 {
  font-weight: 700;
  line-height: 1.2;
  hyphens: none;
  break-after: avoid;
}

h1 { font-size: 1.5rem; margin-top: 2.5rem; }    /* 24px at 16px base */
h2 { font-size: 1.25rem; margin-top: 2rem; }      /* 20px */
h3 { font-size: 1.125rem; margin-top: 1.5rem; }   /* 18px */
```

## Paragraphs and Spacing

**Choose ONE method to separate paragraphs - never both:**

1. **Space between paragraphs** (most common on web): 50-100% of body font size
2. **First-line indent:** 1-4x the font size (no space between paragraphs)

```css
/* Method 1: Space (recommended for web) */
p { margin-bottom: 1em; }           /* = 100% of font size */
p + p { margin-top: 0; }            /* Prevent double spacing */

/* Method 2: Indent (books, long-form) */
p { margin-bottom: 0; }
p + p { text-indent: 1.5em; }       /* Don't indent first paragraph */
```

**Tailwind:** The `prose` class from `@tailwindcss/typography` handles paragraph spacing well out of the box.

## Emphasis (Bold and Italic)

- **Use bold OR italic, never both together.** They are mutually exclusive emphasis tools.
- **Use sparingly.** If everything is emphasized, nothing is.
- **Never emphasize entire paragraphs.**
- **Sans-serif fonts:** Skip italic (the slant is too subtle). Use bold instead.
- **Serif fonts:** Use italic for gentle emphasis, bold for stronger emphasis.
- **Consider alternatives:** Small caps or a size change can work instead of bold/italic.

## All Caps and Small Caps

**All caps is acceptable for less than one line** - labels, short headings, navigation items.

```css
.label {
  text-transform: uppercase;
  letter-spacing: 0.05em;   /* REQUIRED: 5-12% extra spacing */
  font-size: 0.75em;        /* Reduce size slightly */
}
```

**Always add letter-spacing to caps and small caps.** Without it, capital letters look cramped.

**Tailwind:** `uppercase tracking-wide` or `tracking-wider`.

**Small caps via CSS:**
```css
.small-caps {
  font-variant: small-caps;
  letter-spacing: 0.05em;
}
```

Note: `font-variant: small-caps` only works properly if the font includes real small cap glyphs. Otherwise the browser fakes them (badly).

## Letter Spacing and Kerning

**Kerning: always on.** Enable it globally.

```css
body {
  font-kerning: auto;                    /* Usually on by default */
  text-rendering: optimizeLegibility;    /* Enables kerning + ligatures */
}
```

**Letter-spacing rules:**
- **All caps and small caps:** Add 0.05em-0.12em
- **Regular lowercase body text:** Don't touch it (0)
- **Large headlines in lowercase:** Tighten slightly (-0.01em to -0.02em)

## Font Pairing

- **Most projects need at most 2 fonts.** One is often enough with weight/size variations.
- **Few designs tolerate 3 fonts. Almost none can handle 4+.**
- **Same-designer pairings work reliably.** Examples: Inter + Inter Tight, IBM Plex Sans + IBM Plex Serif.
- **Assign each font a distinct role** (e.g., headings vs body) and stick with it.
- **Serif + sans-serif is NOT required.** Low-contrast pairings (two serifs, two sans-serifs) can work equally well.

## Text Alignment

- **Left-aligned is the default and safest choice** for web body text.
- **Justified text requires hyphenation** to avoid ugly word spacing. Browser justification engines are mediocre - use left-align unless you have a good reason.
- **Centered text: use sparingly.** Only for short blocks like headings, pull quotes, or hero text. Never for body paragraphs.

```css
/* If you must justify */
.justified {
  text-align: justify;
  hyphens: auto;          /* Required with justified text */
  -webkit-hyphens: auto;
}
```

## Lists

- Bullets should be proportional - noticeable but not oversized
- List indices can use a smaller font size than list item text
- Use semantic HTML (`<ul>`, `<ol>`) - never fake lists with manual bullets

## Block Quotations

```css
blockquote {
  margin-left: 2em;       /* 2-5em indent */
  font-size: 0.95em;      /* Slightly smaller than body */
  line-height: 1.35;      /* Slightly tighter */
}
```

Don't add quotation marks - the indentation already signals a quote. Use block quotes sparingly.

## Tables

```css
table { font-size: 0.9em; }    /* 90% of body */

td, th {
  padding: 0.5em 0.75em;       /* Generous cell padding */
  border: none;                  /* Start borderless */
}

/* Add only the borders you need */
th { border-bottom: 1px solid #e5e7eb; }
tr + tr { border-top: 1px solid #f3f4f6; }

/* Align numbers for scanning */
td.numeric {
  font-variant-numeric: tabular-nums lining-nums;
  text-align: right;
}
```

**Principle:** Start with no borders, add only where needed for legibility. Borders are guides for building tables - they're less useful once the table is full.

## Rules and Borders

- Thickness: 0.5-1px only. Thinner won't render; thicker creates noise.
- Solid lines only - no dots, dashes, or double lines.
- Headings: place rules above (not below) to separate from previous section.
- Prefer whitespace over rules where possible.

## OpenType Features

Enable useful features when the font supports them:

```css
body {
  font-feature-settings: "kern" 1, "liga" 1;    /* Kerning + ligatures */
}

/* Tabular numbers in tables/data */
.data { font-variant-numeric: tabular-nums lining-nums; }

/* Oldstyle figures in body text (if font supports) */
.prose { font-variant-numeric: oldstyle-nums proportional-nums; }

/* Proper fractions */
.fraction { font-variant-numeric: diagonal-fractions; }
```

## Responsive Typography

**The rules don't change with screen size** - only the values adjust.

- Maintain 45-90 character line length at every breakpoint
- Scale font size with viewport but set min/max bounds
- Use `clamp()` for fluid typography:

```css
body {
  font-size: clamp(16px, 1vw + 14px, 20px);
}
```

**Tailwind responsive:**
```html
<p class="text-base md:text-lg max-w-prose">...</p>
```

**Reading distance matters:** Mobile screens are held closer, so relatively smaller text is still readable. But don't go below 16px on any device.

## Complete Starter Stylesheet

A minimal typography foundation for any web project:

```css
:root {
  --font-body: 'Inter', system-ui, sans-serif;
  --font-heading: var(--font-body);
  --text-color: #374151;
  --heading-color: #111827;
}

body {
  font-family: var(--font-body);
  font-size: clamp(16px, 1vw + 14px, 20px);
  line-height: 1.4;
  color: var(--text-color);
  font-kerning: auto;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
}

h1, h2, h3, h4 {
  font-family: var(--font-heading);
  color: var(--heading-color);
  font-weight: 700;
  line-height: 1.2;
  hyphens: none;
  break-after: avoid;
}

h1 { font-size: 1.5em; margin-top: 2.5em; margin-bottom: 0.5em; }
h2 { font-size: 1.25em; margin-top: 2em; margin-bottom: 0.5em; }
h3 { font-size: 1.125em; margin-top: 1.5em; margin-bottom: 0.5em; }

p, ul, ol, blockquote {
  max-width: 65ch;
  margin-bottom: 1em;
}

blockquote {
  margin-left: 2em;
  font-size: 0.95em;
  line-height: 1.35;
}

table { font-size: 0.9em; }
td, th { padding: 0.5em 0.75em; }

.uppercase, .label {
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
```

## Tailwind Typography Plugin

If using Tailwind, the `@tailwindcss/typography` plugin provides a `prose` class that handles most of these rules. Review and customize its defaults:

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      typography: {
        DEFAULT: {
          css: {
            maxWidth: '65ch',
            color: '#374151',
            lineHeight: '1.4',
            h1: { fontSize: '1.5em', lineHeight: '1.2' },
            h2: { fontSize: '1.25em', lineHeight: '1.25' },
            h3: { fontSize: '1.125em', lineHeight: '1.3' },
          },
        },
      },
    },
  },
}
```

## Punctuation Quick Reference

| Instead of | Use |
|-----------|-----|
| Straight quotes `"` `'` | Curly quotes \u201c \u201d \u2018 \u2019 |
| Two spaces after period | One space |
| `--` or `---` for dashes | \u2013 (en dash) or \u2014 (em dash) HTML: `&ndash;` `&mdash;` |
| `...` (three periods) | \u2026 (ellipsis) HTML: `&hellip;` |
| `'` for apostrophe | \u2019 (curly apostrophe) |
| Multiple exclamation marks | One, maximum. Prefer zero. |

## How to Verify

### Quick Checks
- Body text is 15-25px
- Line height is 1.2-1.45 (not the browser default ~1.2 or word processor 1.5)
- No line exceeds 90 characters (check with browser dev tools)
- Headings are only slightly larger than body, not 2x
- Text color is dark gray, not pure black (on light backgrounds)
- Caps and small caps have letter-spacing

### Visual Test
- Read a full paragraph - is it comfortable?
- Squint at the page - does it have even "color" (consistent gray tone)?
- Check on mobile - is the text at least 16px?
- Look at heading spacing - is there more space above headings than below?

## Source

Rules adapted from Butterick's Practical Typography (practicaltypography.com). For deeper understanding of any rule, consult the original source.
