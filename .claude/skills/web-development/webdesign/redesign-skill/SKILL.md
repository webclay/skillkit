# Redesign Skill: Systematic Upgrade to Premium Quality

## Overview

Outlines a systematic approach to upgrading existing websites and applications to premium quality by identifying and replacing generic AI design patterns while preserving functionality.

## Core Workflow

1. **Scan** - Analyze the codebase to identify frameworks, styling methods, and current patterns
2. **Diagnose** - Conduct a comprehensive design audit listing weaknesses and missing states
3. **Fix** - Apply targeted improvements using the existing technology stack

## Key Audit Categories

### Typography Issues
"Browser default fonts or Inter everywhere" should be replaced with distinctive typefaces like Geist or Outfit. Headlines require increased size, tighter letter-spacing, and reduced line-height for visual weight. Body text should be constrained to approximately 65 characters per line with enhanced line-height for readability.

### Color & Surface Problems
Avoid pure black backgrounds, oversaturated accent colors, and the recognizable "purple/blue AI gradient aesthetic." Maintain single accent colors, consistent gray families, and tinted shadows that match background hues rather than generic black shadows.

### Layout Concerns
"Three equal card columns as feature row" represents the most generic pattern and should be replaced with asymmetric alternatives. Full-screen sections should use `min-height: 100dvh` instead of `height: 100vh` to prevent mobile layout issues.

### Interactivity Requirements
All interactive elements need hover states, active feedback, smooth transitions (200-300ms), visible focus rings, and appropriate loading/empty/error states. Dead links and missing navigation indicators should be eliminated.

### Content Standards
Replace placeholder elements with diverse, realistic data. Avoid clichéd copywriting phrases and maintain consistent, purposeful naming conventions throughout the interface.

## Implementation Priority

Changes should be applied in this sequence for maximum impact:

1. Font selection
2. Color palette refinement
3. Interactive state additions
4. Layout and spacing adjustments
5. Generic component replacement
6. State design completion
7. Typography polish

## Operational Constraints

- Preserve existing technology stacks without framework migration
- Maintain all current functionality during upgrades
- Verify new dependencies exist in project files before importing
- Review changes incrementally rather than implementing complete rewrites
