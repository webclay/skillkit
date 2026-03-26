# Industrial Brutalism & Tactical Telemetry UI

## Overview

Synthesizes Swiss typographic modernism with military/aerospace interface aesthetics. Mandates choosing one visual archetype per project - either Swiss Industrial Print (light) or Tactical Telemetry (dark) - and maintaining strict consistency throughout.

## Core Principles

### Typography Dominance
Text is primary structural and decorative infrastructure. Macro-typography (structural headers) employs heavy neo-grotesque fonts at massive scales with compressed leading and negative letter-spacing. Micro-typography (data) uses monospace typefaces at fixed sizes with generous tracking.

### Color Austerity
"Gradients, soft drop shadows, and modern translucency are strictly prohibited." Each substrate palette consists of three colors maximum: background, foreground, and hazard red (`#E61919` or `#FF2A2A`) as the sole accent. Terminal green is optional for single UI elements only.

### Spatial Rigor
Layouts employ rigid CSS Grid structures with visible 1-2px borders compartmentalizing information zones. Negative space is calculated rather than intuitive. All corners measure exactly 90 degrees - no border-radius.

### Analog Degradation
CRT scanlines, halftone dithering, and mechanical noise filters simulate hardware constraints and physical media imperfections, preventing purely digital appearance.

## Technical Implementation

- Font pairing: Neue Haas Grotesk/Inter Black (headers) + JetBrains Mono/IBM Plex Mono (data)
- Layout: `display: grid; gap: 1px;` for mathematically perfect dividing lines
- Semantic HTML: Prioritize `<data>`, `<samp>`, `<kbd>`, `<output>`, `<dl>` tags
- Scaling: CSS `clamp()` functions for responsive macro-typography
