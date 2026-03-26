# Premium Utilitarian Minimalism: UI Architect Protocol

## Core Philosophy

"Clean editorial-style interfaces. Warm monochrome palette, typographic contrast, flat bento grids, muted pastels."

## Banned Elements

- Generic typefaces (Inter, Roboto, Open Sans)
- Standard icon libraries (Lucide, Feather, Heroicons)
- Heavy shadows, gradients, neon colors, glassmorphism
- Emojis, placeholder content, AI marketing clichés

## Typography Architecture

**Sans-serif body:** SF Pro Display, Geist Sans, Helvetica Neue, Switzer

**Serif headlines:** Lyon Text, Newsreader, Playfair Display, Instrument Serif

**Monospace:** Geist Mono, SF Mono, JetBrains Mono

Off-black text (#111111 or #2F3437), tight tracking, generous line-height (1.6)

## Color System

Warm monochromatic base (white, bone, charcoal) with reserved muted pastel accents (pale red, blue, green, yellow) used sparingly for semantic meaning only.

## Components

Bento grids with 1px borders (#EAEAEA), 8-12px border-radius, generous padding, solid black buttons (#111111), and tag-style badges.

## Motion

Subtle fade-in animations via `IntersectionObserver`, hover scale effects, staggered reveals - prioritizing performance through transform/opacity only.
