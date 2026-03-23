---
name: animation
description: Use this skill when adding animations or transitions to web pages. Activate when the user mentions animations, hover effects, scroll reveals, transitions, motion, fade-in, parallax, carousels, or micro-interactions. Prevents over-animation and enforces tasteful motion defaults.
---

# Web Animation

Opinionated animation rules for web projects. Based on analysis of 200+ SaaS websites. Apply these rules when building or reviewing UI - they work with any CSS framework (Tailwind, vanilla CSS, Framer Motion).

## When to Use This Skill

- Adding animations or transitions to a page
- Building hover effects, scroll reveals, or page transitions
- Reviewing a page for animation quality
- User mentions motion, animations, transitions, or micro-interactions
- Deciding whether something should be animated at all

## Core Principle

Motion should guide attention, not steal it. Every animation must pass this test:

1. Remove the animation. Does the page still make sense? If yes - keep it.
2. Does it guide the user's eye to the CTA? If no - delete it.
3. Does it add more than 0.5s to load time? If yes - delete it.
4. Would you notice if it was gone? If no - it was never needed.

## Anti-Patterns (Never Do These)

- Hero text bouncing in letter by letter
- Parallax scrolling on everything
- Auto-playing carousels
- Floating/pulsing CTA buttons
- Page transitions that take 2+ seconds
- Scroll-jacking (hijacking native scroll behavior)
- Animating every element on scroll
- Bounce or overshoot easing on UI elements

## The 5 Places to Animate

These are the only places that benefit from animation on a typical web page. Everything else should be static.

### 1. Hero Section

Fade in headline + CTA on page load. One animation. Done.

```css
/* CSS */
.hero-content {
  animation: fadeIn 0.4s ease-out;
}

.hero-cta {
  animation: fadeIn 0.4s ease-out 0.1s both;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
```

**Tailwind:** `animate-in fade-in slide-in-from-bottom-2 duration-400`

**Rules:**
- Duration: 0.4s ease-out
- Delay: 0.1s between elements (headline, then CTA)
- Max 2-3 elements animated, not every item in the hero

### 2. Cards and Buttons (Hover)

Subtle lift on hover. Shadow increase. Scale 1.02 max.

```css
/* CSS */
.card {
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.card:hover {
  transform: translateY(-2px) scale(1.02);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
}

.button {
  transition: transform 0.2s ease, background-color 0.2s ease;
}

.button:hover {
  transform: scale(1.02);
}
```

**Tailwind:** `transition-all duration-200 hover:-translate-y-0.5 hover:scale-[1.02] hover:shadow-lg`

**Rules:**
- Duration: 0.2s ease
- Never scale above 1.05 - that looks cheap
- Combine translateY(-2px) with subtle scale for lift effect
- Always transition `transform` and `box-shadow` together

### 3. Scroll Reveals

Sections fade up as they enter the viewport. Once. Not every scroll.

```css
/* CSS */
.reveal {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 0.5s ease, transform 0.5s ease;
}

.reveal.visible {
  opacity: 1;
  transform: translateY(0);
}
```

```js
// Vanilla JS - Intersection Observer
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add('visible');
        observer.unobserve(entry.target); // Once only
      }
    });
  },
  { threshold: 0.2 }
);

document.querySelectorAll('.reveal').forEach((el) => observer.observe(el));
```

**Tailwind (with plugin or custom):** Use Intersection Observer to add classes. Or use a library like `motion` (Framer Motion standalone).

**Rules:**
- Translate Y: 20px max (not 50px or 100px)
- Duration: 0.5s
- Trigger: when 20% visible (threshold: 0.2)
- Fire once - unobserve after revealing
- Never re-animate on scroll up

### 4. FAQ Accordions

Smooth expand/collapse. Rotate the chevron icon. That's it.

```css
/* CSS */
.accordion-content {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 0.3s ease-in-out;
}

.accordion-content.open {
  grid-template-rows: 1fr;
}

.accordion-content > div {
  overflow: hidden;
}

.accordion-chevron {
  transition: transform 0.3s ease-in-out;
}

.accordion-chevron.open {
  transform: rotate(180deg);
}
```

**Rules:**
- Duration: 0.3s ease-in-out
- No bounce. No overshoot.
- Use `grid-template-rows` trick for smooth height animation (not max-height hacks)
- Only animate the chevron rotation and content height

### 5. Form Validation

Green checkmark fades in on valid input. Red shake on error.

```css
/* CSS */
.checkmark {
  opacity: 0;
  transition: opacity 0.2s ease;
}

.input-valid .checkmark {
  opacity: 1;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-3px); }
  75% { transform: translateX(3px); }
}

.input-error {
  animation: shake 0.3s ease;
}
```

**Tailwind (shake):** `animate-[shake_0.3s_ease]` with custom keyframes in config.

**Rules:**
- Shake: 3px horizontal, 0.3s duration
- Checkmark: 0.2s fade
- Keep it minimal - one visual cue per state change

## Quick Reference

| Element | Duration | Easing | Transform |
|---------|----------|--------|-----------|
| Hero fade-in | 0.4s | ease-out | translateY(10px) + opacity |
| Card hover | 0.2s | ease | scale(1.02) + translateY(-2px) |
| Scroll reveal | 0.5s | ease | translateY(20px) + opacity |
| Accordion | 0.3s | ease-in-out | grid-template-rows |
| Form shake | 0.3s | ease | translateX(3px) |
| Checkmark | 0.2s | ease | opacity |

## Performance

- Prefer `transform` and `opacity` - they don't trigger layout recalculation
- Never animate `width`, `height`, `top`, `left`, or `margin` - use `transform` instead
- Use `will-change: transform` sparingly and only on elements that will animate
- Test on low-end devices - if it stutters, simplify or remove

## How to Verify

- Remove all animations. Does the page still work? Good.
- Watch someone use the page. Do they notice the animations? They shouldn't.
- Load the page on a 4G connection. Do animations delay meaningful content? Fix it.
- Check for layout shifts caused by animations (CLS in Lighthouse).

## Source

Rules adapted from Supafast's analysis of 200+ SaaS websites.
