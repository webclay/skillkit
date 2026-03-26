---
name: animated-sections
description: Create scroll-driven animated website sections using AI-generated video assets (Cling 3.0, NanoBanana). Covers hero video backgrounds, scroll-tied frame-by-frame animations, gradient masking, and performance optimization. Use when the user wants animated 3D effects, exploding views, rotating objects, or cinematic scroll sections on a landing page.
---

# Animated Sections - AI Video to Scroll Animation

Turn AI-generated video assets into polished scroll-driven website animations. This skill covers the full workflow: generating the video asset externally, integrating it as a hero background or scroll animation, and optimizing for performance.

## When to Use This Skill

- User wants a 3D animated hero background (rotating object, panning scene)
- User wants a scroll-driven "exploding view" or frame-by-frame reveal
- User mentions Cling 3.0, NanoBanana, Higsfield, or AI-generated video
- User wants cinematic scroll effects on a landing page
- User drops an MP4 file and asks to integrate it into the site

---

## Step 1: Generate the Video Asset (External)

Claude Code does not generate the video. The user creates it on an external platform. Guide them with these prompts when asked:

### Recommended platforms
- **Cling 3.0** (via Higsfield) - best for 3D animations, exploding views, rotating objects
- **NanoBanana 2** - good for generating the source image that gets animated

### Settings for web-ready output
- Duration: 5 seconds (enough frames for a smooth scroll sequence)
- Aspect ratio: 16:9 (landscape, fits hero sections)
- Quality: 1080p
- Background: white or matching the site's background color

### Prompt templates for common effects

**Rotating object (globe, product, spaceship):**
> Generate a high-quality 3D render of a [object] rotating in the exact same place. Center of mass should not move, just rotating perfectly on its axis. White background, no text.

**Exploding/expanding view (house, product, device):**
> Generate a high-quality exploding view animation of a [object] to showcase its components. No text, white background. It should explode in all directions, including vertically and horizontally. None of it should go outside the bounds of the video.

**Panning 3D scene (interior, landscape, cityscape):**
> Generate a high-quality 3D render style video of panning through a [scene description]. Make the background white. Make the asset super high quality. This should look like something you'd see on a premium website or landing page.

### Tips for better results
- Generate 2-3 versions and pick the best one
- Use NanoBanana to generate a source image first, then animate it with Cling 3.0
- Keep the background solid (white or dark) - it masks much easier into the site
- Avoid text in the video - overlay text with HTML instead

---

## Step 2: Hero Video Background

For a looping video background behind the hero section.

### Implementation

```html
<section class="relative min-h-screen overflow-hidden">
  <!-- Video background -->
  <video
    autoplay
    loop
    muted
    playsinline
    class="absolute inset-0 h-full w-full object-cover"
  >
    <source src="/videos/hero-bg.mp4" type="video/mp4" />
  </video>

  <!-- Gradient mask: blends video edges into the page background -->
  <div class="absolute inset-0 bg-gradient-to-t from-white via-white/60 to-white/80"></div>
  <div class="absolute inset-0 bg-gradient-to-r from-white/40 via-transparent to-white/40"></div>

  <!-- Content on top -->
  <div class="relative z-10 flex min-h-screen items-center justify-center">
    <h1 class="text-5xl font-bold">Your headline</h1>
  </div>
</section>
```

### Key details
- Always include `muted playsinline` - required for autoplay on mobile
- Layer multiple gradient overlays (top-bottom AND left-right) for a clean vignette
- Adjust gradient opacity until text is clearly readable
- Match the gradient `from-` color to the page background (e.g., `from-white` or `from-gray-950`)
- Use `object-cover` to fill the section without distortion

### Optional: mouse-tied parallax
Add subtle movement tied to mouse position for a premium feel:

```html
<video id="hero-video" class="... transition-transform duration-300 ease-out">
```

```js
document.addEventListener('mousemove', (e) => {
  const x = (e.clientX / window.innerWidth - 0.5) * 10;
  const y = (e.clientY / window.innerHeight - 0.5) * 10;
  document.getElementById('hero-video').style.transform = `translate(${x}px, ${y}px) scale(1.05)`;
});
```

Scale slightly above 1.0 so the movement doesn't reveal edges.

---

## Step 3: Scroll-Driven Frame Animation

For "exploding view" or frame-by-frame effects that play as the user scrolls.

### How it works
1. Extract all frames from the video as optimized JPEG images
2. Preload the frames
3. Tie the current visible frame to the scroll position
4. Overlay text sections that fade in/out at specific scroll points

### Frame extraction
Use ffmpeg to extract frames from the MP4:

```bash
# Create output directory
mkdir -p public/frames

# Extract frames as optimized JPEGs (quality 80, ~150 frames for 5s video at 30fps)
ffmpeg -i input.mp4 -q:v 4 -vf "fps=30" public/frames/frame-%04d.jpg
```

If 150 frames is too heavy, reduce to 15fps (75 frames) or 10fps (50 frames). Test scroll smoothness vs. load time.

### Implementation

```html
<section id="scroll-animation" class="relative" style="height: 300vh;">
  <!-- Sticky container: stays in view while user scrolls through the 300vh -->
  <div class="sticky top-0 h-screen overflow-hidden">
    <!-- Frame canvas -->
    <canvas id="frame-canvas" class="absolute inset-0 h-full w-full object-contain"></canvas>

    <!-- Gradient mask (same technique as hero) -->
    <div class="absolute inset-0 bg-gradient-to-t from-white via-transparent to-white"></div>
    <div class="absolute inset-0 bg-gradient-to-b from-white/80 via-transparent to-white/80"></div>

    <!-- Scroll-triggered text overlays -->
    <div class="relative z-10 flex h-full items-center justify-center">
      <div id="scroll-text" class="text-center transition-opacity duration-500">
        <h2 class="text-4xl font-bold">Section headline</h2>
        <p class="mt-4 text-lg text-gray-600">Supporting text</p>
      </div>
    </div>
  </div>
</section>
```

```js
const canvas = document.getElementById('frame-canvas');
const ctx = canvas.getContext('2d');
const frameCount = 150;
const frames = [];

// Preload all frames
for (let i = 1; i <= frameCount; i++) {
  const img = new Image();
  img.src = `/frames/frame-${String(i).padStart(4, '0')}.jpg`;
  frames.push(img);
}

// Draw frame based on scroll position
function updateFrame() {
  const section = document.getElementById('scroll-animation');
  const rect = section.getBoundingClientRect();
  const scrollFraction = Math.max(0, Math.min(1,
    -rect.top / (rect.height - window.innerHeight)
  ));
  const frameIndex = Math.min(
    frameCount - 1,
    Math.floor(scrollFraction * frameCount)
  );
  const frame = frames[frameIndex];
  if (frame.complete) {
    canvas.width = canvas.offsetWidth;
    canvas.height = canvas.offsetHeight;
    ctx.drawImage(frame, 0, 0, canvas.width, canvas.height);
  }
  requestAnimationFrame(updateFrame);
}
requestAnimationFrame(updateFrame);
```

### Text overlay timing
Fade text sections in/out at specific scroll percentages:

```js
const textSections = [
  { el: document.getElementById('text-1'), start: 0.0, end: 0.3 },
  { el: document.getElementById('text-2'), start: 0.35, end: 0.65 },
  { el: document.getElementById('text-3'), start: 0.7, end: 1.0 },
];

function updateText(scrollFraction) {
  textSections.forEach(({ el, start, end }) => {
    const visible = scrollFraction >= start && scrollFraction <= end;
    el.style.opacity = visible ? '1' : '0';
  });
}
```

---

## Step 4: Performance Optimization

Video and frame animations are heavy. Always optimize.

### Video compression (hero background)
```bash
# Compress MP4: reduce size by 80-95% while maintaining visual quality
ffmpeg -i input.mp4 -vcodec libx264 -crf 28 -preset slow -vf "scale=1920:-2" -an output.mp4
```

- `-crf 28`: good balance of quality and size (lower = better quality, bigger file)
- `-an`: strips audio (not needed for background video)
- Target: under 500KB for hero video, under 2MB max

### Frame optimization
```bash
# Optimize extracted frames (reduce quality slightly for massive size savings)
for f in public/frames/*.jpg; do
  jpegoptim --max=75 --strip-all "$f"
done
```

### Preloading strategy
- Preload the first 20 frames immediately (above the fold)
- Lazy-load remaining frames as user scrolls closer
- Use `IntersectionObserver` to start preloading when the scroll section enters the viewport

### If it's still laggy
1. Reduce frame count: extract at 10fps instead of 30fps
2. Reduce resolution: scale frames to 1280px wide instead of 1920px
3. Use WebP instead of JPEG for frames (20-30% smaller)
4. Add `will-change: transform` to the canvas element

---

## Gradient Masking Reference

The gradient mask is what makes the animation look integrated rather than pasted on. Adjust these based on background color:

### White background site
```html
<div class="absolute inset-0 bg-gradient-to-t from-white via-white/60 to-white/80"></div>
```

### Dark background site
```html
<div class="absolute inset-0 bg-gradient-to-t from-gray-950 via-gray-950/60 to-gray-950/80"></div>
```

### Rules
- Top and bottom gradients should be stronger (80%+) so sections above/below blend cleanly
- Side gradients can be lighter (40%) unless text runs close to the edges
- If text is hard to read over the animation, increase the gradient or add a text-shadow/backdrop
- Test at multiple scroll positions - the gradient should look seamless throughout

---

## Astro Integration Notes

In Astro projects, these animations require client-side JavaScript:

- Wrap the scroll logic in a `<script>` tag in your `.astro` component
- The `<video>` element works natively in `.astro` files (no island needed)
- The canvas frame animation needs a `<script>` tag (vanilla JS, no framework needed)
- Place video/frame assets in the `public/` directory so Astro serves them as-is
- For the mouse parallax effect, use a `<script>` tag, not a React island

No external JS libraries are required. Vanilla `requestAnimationFrame` + `IntersectionObserver` handles everything.

---

## Quick Checklist

Before calling an animated section done:

- [ ] Video compressed to under 500KB (hero) or frames under 50KB each (scroll)
- [ ] Gradient mask blends smoothly into the page background at top, bottom, and sides
- [ ] Text is clearly readable over the animation at every scroll position
- [ ] `muted playsinline` attributes on any `<video>` element
- [ ] Frames preload progressively (not all at once on page load)
- [ ] Scroll animation feels smooth at normal scrolling speed
- [ ] Mobile: video autoplay works (requires muted), or falls back to a static image
- [ ] No content shifts or layout jank when frames load
