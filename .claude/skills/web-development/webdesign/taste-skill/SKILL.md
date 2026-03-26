# Premium Frontend Design Framework: Taste Skill

## Core Configuration

Baseline aesthetic operates on three dials:
- **Design Variance (8)** - Layout experimentation level
- **Motion Intensity (6)** - Animation quantity
- **Visual Density (4)** - Content concentration per screen

All dynamically adjustable per user request.

## Architecture Standards

- React/Next.js with Server Components as default
- Tailwind CSS for styling (with version verification)
- Mandatory dependency checks before imports
- "Use client" isolation for interactive components

## Design Engineering Rules

### Typography
Emphasizes avoiding generic fonts like Inter, favoring alternatives such as Geist or Outfit. For technical dashboards, serif fonts are explicitly prohibited.

### Color
"Max 1 Accent Color" with saturation below 80%, avoiding the "AI Purple/Blue aesthetic" entirely.

### Layout
Centered hero sections are banned when variance exceeds 4; asymmetric structures are preferred instead.

### Anti-Patterns
Cards are discouraged unless elevation serves hierarchy; data should "breathe" via negative space or dividers.

## Motion & Performance

- Magnetic micro-physics and perpetual animations only when motion intensity warrants
- Hardware acceleration via `transform` and `opacity` exclusively
- Strict cleanup functions in useEffect blocks
- Framer Motion for UI; GSAP/Three.js only for isolated scrolltelling

## Forbidden Patterns

Neon glows, pure black, oversaturated accents, excessive gradient text, generic names ("John Doe"), and broken image links are explicitly prohibited.

## Advanced Paradigms

The skill catalogs 40+ high-end interaction patterns (Bento grids, parallax tilt cards, liquid swipe transitions, kinetic typography) alongside the "Motion-Engine Bento Paradigm" for SaaS dashboards with perpetual micro-interactions.

## Pre-Flight Checklist

Ensures mobile responsiveness, proper state isolation, and complete UI state coverage (loading, empty, error).
