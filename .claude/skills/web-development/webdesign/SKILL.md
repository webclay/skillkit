---
name: webdesign
description: Master web design skill - automatically selects and applies the right design sub-skills based on the task. Use when building, redesigning, or polishing any website UI. Covers design philosophy, Tailwind implementation, and visual style systems.
---

# Web Design - Master Skill

Orchestrates the web design sub-skills so you only need to reference this one file. Read this skill whenever the task involves designing, building, or polishing a website's visual appearance.

## How This Works

Based on what the user is doing, read and apply the relevant sub-skills from this folder. Always apply the rules from every active skill simultaneously.

---

## Skill Selection

### Starting a new site from scratch
Read and apply:
1. **taste-skill** - sets overall design direction (typography, color, layout rules)
2. **ui-polish** - specific Tailwind tricks for production quality

### Polishing or refining an existing site
Read and apply:
1. **ui-polish** - border/shadow fixes, typography tweaks, layout improvements
2. **redesign-skill** - systematic audit and fix workflow

### Full redesign of a client's existing site
Read and apply:
1. **redesign-skill** - scan, diagnose, fix workflow
2. **taste-skill** - design direction for the new version
3. **ui-polish** - implementation quality

### Specific aesthetic requested by user
Read the matching style skill:

| User says... | Read this skill |
|---|---|
| "premium", "luxury", "agency-level", "high-end" | **soft-skill** |
| "clean", "minimal", "editorial", "monochrome" | **minimalist-skill** |
| "brutalist", "industrial", "terminal", "Swiss" | **brutalist-skill** |
| Nothing specific about style | **taste-skill** (default) |

Always combine the chosen style skill with **ui-polish** for implementation.

### Claude is producing truncated or incomplete output
Add **output-skill** to whatever else is active. It enforces full output with no shortcuts.

### Adding animated/3D sections (video backgrounds, scroll animations)
Read and apply:
1. **animated-sections** - full workflow for AI-generated video integration, scroll-driven frame animation, gradient masking, and performance optimization
2. Combine with whatever style skill is active (the animation skill handles the technical integration, the style skill handles surrounding design)

**External asset generation required:** The user creates the video on Cling 3.0 (via Higsfield) or NanoBanana. The skill includes prompt templates to guide them. Claude Code handles integration and optimization only.

### Using Google Stitch
Read **stitch-skill** - it generates DESIGN.md files in Stitch's semantic format.

---

## Astro Compatibility

These skills were written with React/Next.js examples. When working on an Astro project (like the astro-payload content website template), apply these translations:

| Skill says... | In Astro, do this instead |
|---|---|
| React Server Components | Regular `.astro` components (server-rendered by default) |
| `"use client"` directive | `client:load` or `client:visible` on interactive islands |
| `useEffect` for cleanup | `<script>` tag in `.astro` file, or `client:load` on a framework component |
| Framer Motion | CSS animations/transitions, or a `client:load` React island for complex motion |
| Next.js `app/` router patterns | Astro `src/pages/` file-based routing |
| `useState`, `useRef` | Only inside `client:load` React/Svelte islands, never in `.astro` files |

**What works identically in Astro:**
- All Tailwind CSS classes and utilities
- All typography techniques (variable Inter, tracking, ch units, text-balance)
- All border/shadow/ring techniques
- All layout patterns (split hero, canvas grid, inline headings)
- All color and spacing approaches
- Everything from ui-polish (pure Tailwind, no framework dependency)

---

## Sub-Skill Reference

| Skill | Folder | Purpose |
|---|---|---|
| **taste-skill** | `taste-skill/` | Design system foundation - typography, color, layout, motion rules, anti-patterns |
| **ui-polish** | `ui-polish/` | Concrete Tailwind tricks - ring borders, variable fonts, split heroes, canvas grids, button alignment |
| **redesign-skill** | `redesign-skill/` | Systematic audit + fix workflow for upgrading existing sites |
| **soft-skill** | `soft-skill/` | Premium/luxury aesthetic - agency-level depth, cinematic rhythm |
| **minimalist-skill** | `minimalist-skill/` | Clean editorial style - warm monochrome, bento grids, muted pastels |
| **brutalist-skill** | `brutalist-skill/` | Swiss typography + terminal/military aesthetic (beta) |
| **output-skill** | `output-skill/` | Prevents truncated output - enforces complete, production-ready code |
| **animated-sections** | `animated-sections/` | AI video to scroll animation - hero backgrounds, frame-by-frame scroll, gradient masking, performance |
| **stitch-skill** | `stitch-skill/` | Google Stitch semantic design system format |
