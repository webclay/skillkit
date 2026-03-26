# Design System: Taste Standard - Complete Reference

## Configuration Dials

The system operates on four adjustable parameters (Creativity, Density, Variance, Motion Intent), each scaling 1-10. Default settings: Creativity `8`, Density `4`, Variance `8`, Motion Intent `6`. Users customize these dials before implementation to match project requirements.

## Visual Theme

A "restrained, gallery-airy interface with confident asymmetric layouts and fluid spring-physics motion" creates an atmosphere described as "clinical yet warm." The design balances elevation through subtle shadowing and emphasizes that "every element earns its place through function."

## Color System

**Core palette:** Canvas White (#F9FAFB), Pure Surface (#FFFFFF), Charcoal Ink (#18181B), Steel Secondary (#71717A), Muted Slate (#94A3B8), with whisper borders and diffused shadows.

**Accent options:** Emerald Signal, Electric Blue, Deep Rose, or Amber Warmth - select one per project. Banned colors include purple neon gradients, pure black, and oversaturated accents exceeding 80% saturation.

## Typography

Approved display fonts: Geist, Satoshi, Cabinet Grotesk, Outfit (Inter explicitly banned for premium contexts). Body copy uses the same family at weight 400 with 1.65 line height and 65-character maximum width. Monospace reserved for code and metadata.

## Components

**Buttons:** Flat surfaces with accent fill, active state via subtle translateY or scale.

**Cards:** 2.5rem rounded corners, pure white fill, whisper border, diffused shadow, 2-2.5rem padding.

**Forms:** Labels positioned above inputs; error text in Deep Rose below.

**Navigation:** Sticky, sleek horizontal layout.

**Loaders:** Skeletal shimmer only - circular spinners banned.

## Hero Section Rules

Signatures include inline image typography embedding contextual visuals between words. "Text must never overlap images or other text." Asymmetric structures are required; centered Heroes are banned. Maximum one primary CTA; "Scroll to explore" and scroll arrows explicitly prohibited.

## Layout & Responsiveness

CSS Grid mandatory; flexbox percentage math banned. All layouts collapse to single-column below 768px. Horizontal scroll on mobile is a "critical failure." Responsive testing required at 375px, 768px, and 1440px viewports. Touch targets minimum 44px.

## Motion Intent

Spring-based physics exclusively (stiffness: 100, damping: 20). Perpetual micro-loops on dashboard components. Staggered orchestration for lists and grids. Hardware rule: animate only `transform` and `opacity`, never positional properties.

## Anti-Patterns (Banned)

No emojis, Inter font, generic serifs, pure black, neon glows, overlapping text-image layers, 3-column equal card layouts, centered Heroes, filler UI text ("Scroll to explore"), fake round numbers, AI clichés ("Seamless," "Unleash"), broken Unsplash links, `h-screen`, or circular loading spinners.
