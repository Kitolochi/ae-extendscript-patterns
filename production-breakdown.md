# Production Breakdown: Luma Uni-1 Launch Video

Shot-by-shot reverse engineering of how each visual element was built. Deduces the tools, layer stacks, compositing approaches, and effect chains behind each technique. Written for replication in After Effects via ExtendScript.

**Source**: [@LumaLabsAI](https://x.com/LumaLabsAI) Uni-1 launch video, Mar 23 2026, 60 seconds, 60fps

---

## Shot 1: Color Festival Title (t=0-2)

**What we see**: "UNI-1" text visible through a real-world color powder explosion (Holi festival footage). Hands visible in the lower frame throwing colored powder.

### How it was made

**This is AI-generated video, not stock footage.** The camera angle, powder dynamics, and text integration are too perfect for a live shoot. Luma generated this with their own model, using a prompt like "aerial view of color festival crowd throwing powder with UNI-1 written on brick wall."

The text appears physically embedded in the scene (painted/stenciled on a wall behind the crowd). Two possible approaches:

1. **Model-native text** (most likely): The generation model placed the text as part of the scene. Luma's own model handles text rendering in generated video. The slightly rough, stencil-like quality of the letterforms confirms this.
2. **Post-composited text** (less likely): Text layer with a brick wall texture alpha matte, placed behind a color powder footage layer with luminance keying. The powder obscuring the text edges argues against this.

### AE Replication

```
Layer stack (top to bottom):
1. Color powder particles (shape layers, ADD blend, animated position)
2. Hands/crowd silhouettes (if needed, generated or stock)
3. Text layer with stone/wall texture applied via track matte
4. Textured wall background

Key effects:
- Text: Bevel Alpha for carved look, Fractal Noise for surface texture
- Powder: Shape layer ellipses with high blur, ADD blend, random drift
- Overall: Turbulent Displace for organic movement, Camera Lens Blur for depth
```

See [title-treatments.md](title-treatments.md) for embossed text patterns.

---

## Shot 2: Embroidered Folk Art (t=1.5-2.5)

**What we see**: "UNI-1" stitched into a traditional folk-art textile with birds, flowers, and geometric border patterns. Camera holds steady with slight texture movement.

### How it was made

**AI-generated image/video.** The stitching shows consistent thread texture across the entire frame, and the folk-art motifs (birds, tulips, geometric borders) follow traditional Kantha embroidery patterns. The model generated a "folk art embroidery with UNI-1 text" and the result was used as a 2-3 second hold with subtle camera movement.

The slight camera drift (0.5-1px/frame) was added in post using AE's Transform effect or a subtle position expression. The texture movement (thread catching light) may be part of the generation or added via Fractal Noise overlay at very low opacity.

### AE Replication

```
Layer stack:
1. Adjustment layer: slight vignette + warmth grade
2. Fabric texture overlay (Fractal Noise, OVERLAY blend, 5% opacity)
3. Main image (generated or photographed embroidery)

Camera motion:
- Transform effect with 0.5px/frame drift on X
- Slight scale 100→101% over 2 seconds
- This creates a "macro lens" examination feel
```

### Production Technique: "Texture Vignettes"

These opening shots (t=0-2.5) establish a pattern: show the product name embedded in different physical media. Each holds 1-1.5 seconds. The purpose is to associate the brand with craftsmanship before showing the product. The hard cut to black (t=2.5) creates a clean break between the "artistic" prologue and the "technical" main body.

---

## Shot 3: Dark Text Card (t=3-4.5)

**What we see**: "First unified intelligence model" in cyan text, centered on a near-black background with subtle blue gradient in upper-right.

### How it was made

**Standard AE text composition.** This is a title card, the simplest element in the video. The dark background has a radial gradient (not pure black) with a blue bias in the upper-right quadrant.

### Construction

```
Layer stack:
1. Text layer: "First unified intelligence model"
   - Font: Helvetica Neue or similar geometric sans
   - Size: ~28px, tracking 100
   - Color: [0.4, 0.85, 0.95] (light cyan)
   - Position: frame center (960, 540)
   - Fade in: opacity 0→100 over 12 frames
2. Background: Dark solid with Ramp effect
   - Start: [0.04, 0.06, 0.10] (dark blue-black)
   - End: [0.02, 0.02, 0.04] (near-black)
   - Ramp positioned upper-right to lower-left
```

### Production Technique: "Cyan on Dark"

The color cyan [0.4, 0.85, 0.95] on near-black [0.03, 0.03, 0.05] is the video's signature palette. Used for narration text, particle highlights, and UI accent. This specific shade reads as "technological" and "cold intelligence." It contrasts with the warm magenta used for the sphere's right hemisphere.

---

## Shot 4: Particle Band (t=4.5-5.5)

**What we see**: White/cyan dots arranged in a horizontal band across frame center. Subtitle reads "They built intelligence in pieces." Dots drift and jostle.

### How it was made

**400-600 shape layer dots** with constrained Y positions (center ± 100px) and random X positions across the full frame width. Each dot has:
- Random size: 2-6px
- Random drift: sin-wave X oscillation (amplitude 10-30px, period 2-4 seconds)
- Y position: 440-640 (narrow band)
- Opacity: 40-90% random per dot

The "pieces" metaphor is literal here: intelligence shown as scattered, disconnected dots.

### How the dots move

Each dot's position expression uses independent sin waves with different phases and frequencies. The jostling effect comes from combining two sin waves (low frequency for drift, high frequency for jitter):

```js
'var x=' + startX + '+Math.sin(f*0.008+' + phase1 + ')*25'
    + '+Math.sin(f*0.04+' + phase2 + ')*3;'  // jitter layer
```

The two-frequency approach prevents dots from feeling like they're on a single synchronized oscillation.

See [particle-systems.md](particle-systems.md) for the full band → burst → sphere formation sequence.

---

## Shot 5: Radial Burst (t=5.5-7)

**What we see**: Dots explode outward from center in a radial pattern. Visible spiral structure (like a galaxy). Frame fills with thousands of dots. Blue-white color. Empty void at center.

### How it was made

**Particle explosion from the band's center point.** Each dot's position transitions from its band position to a radial target calculated using polar coordinates with a spiral component.

### The spiral structure

The spiral is created by adding an angle offset proportional to radius:

```js
var targetAngle = Math.random() * Math.PI * 2;
var targetRadius = 50 + Math.random() * 800;
// Spiral: angle increases with radius
var spiralAngle = targetAngle + targetRadius * 0.003;
var targetX = 960 + targetRadius * Math.cos(spiralAngle);
var targetY = 540 + targetRadius * Math.sin(spiralAngle);
```

The `0.003` spiral factor creates ~1.5 full rotations from center to edge. Stronger values (0.005+) create tighter spirals. The void at center (no dots within 50px of center) is natural because no target positions are generated at radius < 50.

### The explosion timing

The burst happens over ~1 second (60 frames). Dots closer to center reach their targets faster (shorter distance). Dots at the edges are still arriving when the next phase begins, creating overlap between phases.

```
Easing: ease-out cubic (fast start, slow arrival)
Duration per dot: 30-60 frames (proportional to distance)
Dot count: increases from ~400 to ~800 (new dots spawn during burst)
```

### Production Technique: "Phase Overlap"

None of the transitions in this sequence are clean start/stop. Phase 2 (burst) begins while Phase 1 (band) dots are still drifting. Phase 3 (convergence) begins while outer Phase 2 dots are still expanding. This overlap creates continuous motion with no dead frames.

---

## Shot 6: Convergence to Nebula (t=7-9)

**What we see**: Dots collapse inward toward center. A bright white/cyan glow forms at the focal point. A few large colored dots (cyan, magenta) visible near the core. Background dots become sparser.

### How it was made

**Reverse of the burst**, but with different easing (ease-in cubic: slow start, fast finish). The convergence target is a single point at frame center (960, 500). As dots approach the center, three things happen simultaneously:

1. **Dot opacity decreases** as they get close (they dissolve into the core glow)
2. **Core glow increases** (additive white shape layer with 60px Gaussian blur)
3. **Accent dots appear** (2-3 large dots, 15-20px, cyan and magenta, floating near the core)

### Core glow construction

The core is NOT a particle. It's a separate shape layer:

```
Layer: "Core Glow"
- Shape layer ellipse, 80px diameter
- Fill: near-white [0.9, 0.95, 1.0]
- Blend mode: ADD
- Gaussian Blur: 60px
- Opacity: ramps from 0 to 80% over 2 seconds
```

The ADD blend mode makes the glow interact with overlapping dots (they brighten where they cross the glow), creating the nebula effect. Using SCREEN would clip too early. Normal blend mode wouldn't create the additive brightening.

### The accent dots

Two colored dots (one cyan, one magenta) at ~15px size orbit slowly near the core. These are the first hint of the dual-color scheme that the sphere will display. They preview the color palette before the audience sees the full sphere.

```
Orbit radius: 30-50px from center
Orbit speed: 0.015 rad/frame (~52 deg/sec)
Size: 12-18px
No blur (sharp edges to contrast with the soft core)
```

---

## Shot 7: Sphere Formation (t=9-12)

**What we see**: Dots arrange themselves on the surface of a sphere. Left hemisphere cyan, right hemisphere magenta. The sphere sits at frame center, slightly above vertical center. Dots show clear latitude/longitude distribution (not random scatter). Subtitle: "This is the era of Unified Intelligence."

### How it was made

**Spherical coordinate mapping.** Each dot is assigned a position on a sphere surface using theta (longitude) and phi (latitude). The 3D position is projected to 2D by discarding the Z component but using it for size and opacity modulation.

### The color gradient

The cyan-to-magenta gradient follows the sphere's X-axis, not a simple left/right split:

```
Color at position X:
  t = (dotX - leftEdge) / sphereDiameter    // 0 at left, 1 at right
  R = lerp(0.2, 0.85, t)                     // cyan→magenta red channel
  G = lerp(0.8, 0.2, t)                      // high→low green
  B = lerp(0.9, 0.6, t)                      // both stay high-ish
```

This produces a continuous gradient, not a hard boundary. The center dots are a blue-purple midpoint color.

### Latitude lines

The dots are NOT randomly distributed on the sphere surface. They follow visible latitude bands (horizontal rings) with slight randomization. This was done by quantizing the phi angle:

```js
// Quantized latitude bands
var band = Math.floor(Math.random() * 12);  // 12 bands
var phi = (band + Math.random() * 0.3) * Math.PI / 12;
// Longitude is fully random within each band
var theta = Math.random() * Math.PI * 2;
```

This creates the distinctive "wireframe globe" look. Uniform random distribution (`acos(2*random-1)`) looks like a fuzzy ball. Latitude-banded distribution reads as a structured, intentional sphere.

### Depth sorting

Back-hemisphere dots (negative Z) render at lower opacity and smaller size. Front-hemisphere dots are brighter and larger. This creates the illusion of a 3D sphere without actual 3D layers:

```
Front (Z > 0): opacity 80-100%, size 4-6px
Side (Z ≈ 0): opacity 50-70%, size 3-4px
Back (Z < 0): opacity 20-40%, size 2-3px
```

See [particle-systems.md](particle-systems.md) for full sphere implementation.

---

## Shot 8: Sphere Rotation + Title (t=12-14)

**What we see**: Sphere rotates slowly clockwise. "UNIFIED INTELLIGENCE" appears in large white text behind the sphere (text is partially occluded by the sphere). Background shows subtle blue light leak upper-left.

### How it was made

**Two-layer title approach**:

1. Text layer BEHIND the sphere (lower in layer stack)
2. Sphere particles ABOVE the text
3. The text shows through the gaps between dots, creating partial occlusion

The text does NOT use a track matte. The dots are small enough (2-6px) that the text is readable between them. The sphere's natural density (400-600 dots on the visible hemisphere at any moment) provides ~30-40% coverage, leaving 60-70% of the text visible.

### Title treatment

```
Font: Geometric sans-serif (likely Neue Montreal or similar)
Size: ~100px
Tracking: 300+ (very wide, characteristic of tech branding)
Color: white [1, 1, 1]
Position: frame center, vertically aligned with sphere center
Blend mode: NORMAL (no special compositing)
```

### Background light leak

A subtle blue radial gradient in the upper-left quadrant. Created with a shape layer ellipse (600px, 40% opacity, 200px blur, SCREEN or ADD blend). This prevents the background from being flat black and adds atmospheric depth.

### Rotation mechanics

The sphere rotates at ~12 deg/sec (0.0035 rad/frame). This is a 2D rotation transform applied to the particle group, rotating each dot's position around the sphere center:

```
For each dot:
  dx = x - centerX
  dy = y - centerY
  newX = dx * cos(angle) - dy * sin(angle) + centerX
  newY = dx * sin(angle) + dy * cos(angle) + centerY
```

Back-hemisphere dots that rotate to the front gain opacity. Front dots that rotate to the back lose opacity. This creates the illusion of continuous surface.

---

## Shot 9: Sphere Dispersal + Split Composition (t=14-17)

**What we see**: Sphere dissolves into a loose particle cloud occupying the right 2/3 of frame. "UNI-1" text appears in the left 1/6 of frame. Particles are larger than sphere dots (5-15px) and multi-colored (cyan, green, white, magenta).

### How it was made

**Two simultaneous transitions**:

1. Sphere dots scatter to random positions in the right 2/3 of frame (smoothstep easing over 2 seconds)
2. Title text fades in on the left

### Split composition geometry

```
Left zone (text):  x: 100-380, vertical center
Right zone (cloud): x: 550-1820, y: 150-900

Text: "UNI-1"
Font: ~160px, white, tracking 100
Position: (320, 540)
```

The narrow left zone (1/6 of frame) forces the text into a compact, bold placement. The asymmetry (small text zone, large visual zone) reads as confident design. A 50/50 split would feel indecisive.

### Particle size increase

During dispersal, dots grow from sphere size (2-6px) to cloud size (5-15px). Combined with the dots spreading apart, this creates a "camera pushing in" feel without any actual camera movement. The larger dots also allow the multi-color palette to be more visible.

### Color diversification

On the sphere, dots were limited to cyan-magenta gradient. In the cloud, the palette expands:
- Cyan [0.2, 0.8, 0.9]
- Magenta [0.85, 0.2, 0.6]
- White [0.9, 0.9, 0.9]
- Green [0.3, 0.8, 0.5]
- Amber [0.9, 0.6, 0.2] (subtle, few dots)

This represents "unified" intelligence: more diverse than the dual-color sphere.

See [composition-structure.md](composition-structure.md) for split composition safe zones.

---

## Shot 10: Product UI Card (t=17-22)

**What we see**: Light-themed UI card (white background, rounded corners) with thumbnail images, a text input field with "C|" blinking cursor, the prompt being typed at ~47 chars/sec, a blue send button, and eventually a "thinking" animation.

### How it was made

This is a **screen recording of their actual product UI**, composited into the video. The evidence:

1. Pixel-perfect UI chrome (rounded input field, exact iOS-style send button)
2. Scroll physics match native WebKit/Blink scrolling
3. Thumbnails are real photos (Golden Gate Bridge, document icon)
4. The cursor blink rate matches browser default (530ms cycle)

### Compositing approach

```
Layer stack:
1. Adjustment layer: subtle vignette at edges
2. UI screen recording (alpha channel or luminance keyed from white bg)
3. Gradient background: light gray top → slightly darker bottom
   [0.88, 0.88, 0.88] → [0.82, 0.82, 0.82]
```

### Camera work on the UI

The UI card enters from below (80px upward slide, 24 frames). During the typing sequence, the "camera" slowly zooms into the card (scale 100→104% over 4 seconds). When the prompt is submitted, the card scrolls up to show the generation result, simulating native scroll behavior.

### Production Technique: "Screencast Compositing"

Rather than rebuilding the UI in AE, record the actual product and composite it. Benefits:
- Perfect pixel accuracy (no approximation of hover states, scroll physics, font rendering)
- Updates automatically when the product changes
- Authenticated states (real data, real thumbnails) look convincing

For AE replication without screen recording, see [ui-mockups.md](ui-mockups.md).

### Typing speed analysis

The prompt "Create infographics explaining the architecture of the Golden Gate Bridge" (70 characters) types in ~1.5 seconds = 47 chars/sec. This speed communicates "machine input" rather than "human typing." The cursor still blinks during typing (every 15 frames), which is non-standard for real text input but reads better in video.

---

## Shot 11: Generation Visualization (t=28-33)

**What we see**: "Generating..." text in upper-left. The frame shows a partially-rendered image with noise, horizontal scan lines, and cyan light streaks. The image progressively resolves from noise to clarity over ~3 seconds.

### How it was made

**Multi-layer noise-to-clean transition.** The "generating" effect layers multiple techniques:

### Layer stack (top to bottom)

```
1. "Generating..." text (upper-left, small sans-serif, animated dots)
2. Glitch strips (horizontal displacement, cyan/magenta, intermittent)
3. Scan line overlay (2px horizontal line sweeping top to bottom)
4. Noise overlay (Fractal Noise at high contrast, fading out)
5. Clean final image (opacity ramping 0→100)
6. Black background
```

### Noise-to-clean transition

The noise overlay uses Fractal Noise with animated Evolution (creates moving grain). Its opacity decreases from 80→0 over 90 frames while the clean image's opacity increases from 0→100 over the same period. The overlap (both partially visible) creates the "resolving" look.

### The cyan light streaks

Diagonal bright lines that sweep across the frame during generation. These are shape layer paths (2-3px width, white fill, ADD blend, 60% opacity, Gaussian blur 8px) with animated position:

```
Path: diagonal from lower-left to upper-right
Position expression: slides from left to right over 30 frames
Opacity: appears for 20 frames, fades out
Multiple instances with staggered timing
```

### Center-first resolution

The center of the image clears before the edges. Achieved by using a radial gradient as a matte for the noise layer: center of the gradient is transparent (noise disappears here first), edges are opaque (noise lingers).

```
Noise layer matte: Radial gradient ramp
  Center: fully transparent at f=60
  Edge: fully transparent at f=120
  This creates progressive reveal from center outward
```

See [generation-visualization.md](generation-visualization.md) for complete implementation.

---

## Shot 12: Generated Infographic (t=33-37)

**What we see**: Multi-panel infographic layout titled "UNI-1 Golden Gate Infographic." Orange/amber wireframe drawing of the bridge on charcoal background. Labels like "SUSPENSION SYSTEM", "CAB", "DIAM", numerical readouts "27,572". Side panels with additional data. Slow horizontal camera pan reveals content progressively.

### How it was made

**This is output from Luma's own model** — the "generated infographic" that resulted from the prompt typed in Shot 10. It showcases the model's ability to produce structured visual content (not just photos/video).

The infographic itself has these characteristics that identify it as AI-generated:
- Wireframe style is consistent but not architecturally accurate
- Numerical values (27,572) appear decorative rather than factual
- Typography mixing (some labels are clean, some have rendering artifacts)
- Layout follows infographic conventions but with AI-typical inconsistencies

### Compositing the pan

The full infographic is wider than 1920px (estimated 3000-4000px wide). A horizontal pan reveals it:

```
Layer: Generated infographic image (oversized)
Anchor point: left edge
Position expression:
  'var f=timeToFrames(time);'
  'var x=linear(f, 0, DUR, 960, -800);'   // slides left
  '[x, 540]'

Pan speed: ~40px/sec (slow enough to read labels)
Duration: 4 seconds
```

### The wireframe drawing style

If replicating in AE, the orange wireframe can be built with shape layer paths:

```
Stroke color: [1.0, 0.4, 0.0] (amber/orange)
Stroke width: 1-2px
Fill: none (wireframe = stroke only)
Background: [0.1, 0.1, 0.1] (charcoal)
Rounded corners on panels: 6px
```

### Annotation labels

All labels use the same technical typography:
- Font: monospace (Consolas, SF Mono, or similar)
- Size: 12-14px
- Tracking: 200+ (extra wide)
- Color: white [0.9, 0.9, 0.9]
- Case: ALL CAPS
- Leader lines: 1px white, 60% opacity

See [hud-overlays.md](hud-overlays.md) for leader line and numerical readout patterns.

---

## Shot 13: Full-Bleed Generated Content (t=38-52)

**What we see**: Series of full-frame generated images and videos — landscapes, atmospheric scenes, portraits. Each holds for 2-4 seconds with subtle camera movement (drift, zoom). Demonstrates the breadth of the model's generation capability.

### How it was made

**Direct output from their model, composited with minimal post-processing.** Each generated clip runs 2-4 seconds with:

1. Slow zoom (scale 100→103% over the clip duration)
2. Subtle position drift (1-2px/frame in a consistent direction)
3. Cross-dissolve between clips (18-frame overlap)

### The "Ken Burns" approach

Every generated clip uses the same camera formula:

```
Scale expression:
  'var f=timeToFrames(time-inPoint);'
  'var dur=timeToFrames(outPoint-inPoint);'
  'var s=linear(f, 0, dur, 100, 103);'
  '[s,s]'

Position expression:
  'var f=timeToFrames(time-inPoint);'
  'var dur=timeToFrames(outPoint-inPoint);'
  'value + [f*1.5, f*0.5]'     // drift direction varies per clip
```

The 3% zoom over 3 seconds is imperceptible as zoom but prevents the still from feeling static. Combined with the position drift, it suggests a camera operator, not a slide show.

### Transitions between clips

Cross-dissolve with 4-frame overlap:

```
Outgoing clip: opacity 100→0 over 18 frames
Incoming clip: opacity 0→100 starting 4 frames before outgoing finishes
```

The 4-frame overlap prevents dip-to-black. Without it, there's a visible dark frame between clips.

See [composition-structure.md](composition-structure.md) for cross-dissolve patterns.

---

## Shot 14: Content Mosaic (t=53-55)

**What we see**: Irregular grid of 12-15 image tiles. Mixed styles: portraits, illustrations, manga, 3D renders, photography. Some tiles larger than others. Slight rotation on some tiles. Barely perceptible parallax drift.

### How it was made

**Pre-composed tile grid** with staggered entrances. Each tile is a shape layer (rounded rect) with a generated image placed inside via track matte or simply positioned.

### Grid construction

The layout uses intentional size hierarchy, not a uniform grid:

```
Large tiles (2-3):  ~300x250px
Medium tiles (4-6): ~180x180px
Small tiles (6-8):  ~120x120px

Gaps: 8-12px
Tile rotation: ±2-3 degrees (random per tile, not all rotated)
Corner radius: 4-8px
```

### Staggered entrance

Tiles appear in a cascade, 4 frames apart:

```
Tile 1: fade in at f0-f12, scale 90→100%
Tile 2: fade in at f4-f16, scale 90→100%
Tile 3: fade in at f8-f20, scale 90→100%
...
Full cascade for 15 tiles: 72 frames (1.2s)
```

### Parallax drift

Each tile drifts at a slightly different rate, creating depth:

```
Large tiles: 0.2px/frame (slower = farther)
Small tiles: 0.4px/frame (faster = closer)
Direction: consistent (all drift left, or all drift up-left)
```

This parallax effect, even at sub-pixel rates, makes the grid feel like a 3D space rather than a flat arrangement.

### Production Technique: "Social Proof Mosaic"

The mixed content styles (photo, illustration, manga, 3D) serve a specific marketing purpose: demonstrating range. A mosaic of similar-looking outputs would suggest a narrow model. The style diversity says "this model handles everything."

The overlapping tiles (some edges hidden behind others) create casual, organic energy. Rigid alignment reads as corporate. Overlap reads as abundant, overflowing creativity.

---

## Shot 15: Bookend Return (t=55-57)

**What we see**: Particle field from the opening returns. Loose cyan and magenta dots scattered across frame. Subtitle: "This is UNI-1." The particles are in their relaxed state (no formation, no convergence).

### How it was made

**Same particle precomp from Shot 4-8**, placed again at the end with different timing. The particles are in their "band scatter" initial state, not the formed sphere.

### Bookend psychology

The opening particle sequence runs 6 seconds (t=3-9) with full formation arc. The closing runs 2 seconds (t=55-57) in relaxed state. The audience recognizes the visual motif without needing the full sequence replayed. Shorter is better here — it says "you know what this is."

### Overlay comparison

```
Opening (t=4):  400+ dots, tight band, high density, subtitle narration
Closing (t=56): 200+ dots, loose scatter, lower density, subtitle "This is UNI-1."
```

The lower dot count and looser arrangement create a calm, resolved version of the opening tension. The narrative arc: scattered → unified → returned to scatter with new meaning.

See [composition-structure.md](composition-structure.md) for bookend implementation.

---

## Shot 16: Final Title (t=57-59)

**What we see**: "ONLY FROM LUMA" in white text, centered, on a dark background with faint atmospheric gradient. Holds for 2 seconds.

### How it was made

**Simple title card with strategic typography choices.**

```
Text: "ONLY FROM LUMA"
Font: Geometric sans-serif, medium weight
Size: ~60px
Tracking: 400+ (extremely wide — this is a confidence signal)
Color: white [1, 1, 1]
Position: frame center (960, 540)
Fade in: 12 frames
Background: same dark gradient as Shot 3
```

### Production Technique: "Authority Spacing"

The extremely wide tracking (400+ thousandths of an em) on the final title is a deliberate luxury brand technique. Wide tracking says "I don't need to fill the space." It's the typographic equivalent of negative space in design. Combined with ALL CAPS and a geometric sans-serif, it reads as premium and authoritative.

The phrase "ONLY FROM" is marketing exclusivity language. Paired with the wide tracking, it positions Luma as the sole provider of this capability.

---

## Shot 17: Logo Hold (t=59-60)

**What we see**: Luma logo (flame/crystal icon) centered on black. The logo appears to have a subtle 3D rotation or animation (slight perspective shift).

### How it was made

**Logo layer with subtle animation.** The icon has a very slight rotation (±3 degrees) or scale pulse that prevents it from feeling like a dead freeze frame.

```
Logo:
- SVG or PNG import
- Position: frame center
- Scale: 100%
- Subtle animation: either
  (a) 3D rotation: rotationY 0→5→0 degrees over 1 second
  (b) Scale pulse: 98→100% over 0.5s, ease-out
- Background: pure black [0, 0, 0]
```

### Hold duration

One second of logo hold. Two seconds would feel slow. Half a second would feel rushed. One second is the industry standard for end-card logo holds in launch videos.

---

## Cross-Cutting Production Techniques

### 1. Subtitle System

Subtitles appear throughout the video at consistent placement:
- Position: lower-fifth (y ≈ 920)
- Font: sans-serif, ~22px, white
- Tracking: 50 (slight letter-space)
- Horizontal line: left side, ~200px wide, 1px, white 40%, aligns with subtitle baseline
- Fade timing: 12-frame fade in, 12-frame fade out
- No background bar (text sits directly on dark content)

The horizontal line is a distinctive design touch. It anchors the subtitle to the frame edge and signals "narration" rather than "dialogue."

### 2. Pacing Architecture

```
Section              Duration  Shots  Avg shot length
Opening texture      2.5s      2      1.25s
Particle narrative   12s       5      2.4s
Product UI demo      5s        1      5s (continuous)
Generation showcase  10s       1      10s (continuous)
Generated content    17s       4      4.25s
Mosaic + closing     5s        3      1.7s
End cards            3s        2      1.5s
```

The video alternates between short, punchy shots (texture vignettes, mosaic) and long, continuous sequences (UI demo, generation). This rhythm prevents both monotony and exhaustion.

### 3. Color Palette

```
Primary:   Cyan     [0.2, 0.8, 0.9]   — intelligence, technology
Secondary: Magenta  [0.85, 0.2, 0.6]  — creativity, generation
Accent:    Amber    [1.0, 0.4, 0.0]   — data, engineering
Neutral:   Dark bg  [0.03, 0.03, 0.05] — near-black with blue bias
Text:      White    [1.0, 1.0, 1.0]   — titles and narration
```

Every visual element pulls from this palette. The particle sphere maps cyan→magenta. The UI card uses white/gray (neutral). The infographic wireframe uses amber. The consistency creates visual cohesion across different shot types.

### 4. Sound Design Implications

While we can't hear the audio from frame captures, the visual timing reveals the music structure:

- **Beat drops** at t=6 (burst), t=9 (convergence), t=12 (sphere reveal), t=33 (infographic reveal)
- **Build-ups** during particle formation (t=4-9) and generation visualization (t=28-33)
- **Sustained tones** during UI demo (t=17-22) and generated content (t=38-52)

Each visual transition aligns with an audio event. When building this in AE, use markers at these timestamps to sync visual transitions to music beats.

### 5. Camera Language

```
Shots 1-3:    Static (title cards, holds)
Shots 4-8:    "Virtual camera" via particle motion (no actual camera)
Shots 10:     Slow zoom into UI card (scale expression)
Shot 12:      Horizontal pan across infographic
Shots 13:     Ken Burns (drift + zoom) per clip
Shot 14:      Parallax layers (simulated depth)
Shots 15-17:  Static (closing holds)
```

The "camera" never moves during particle sequences. The motion comes from the particles themselves. This is cheaper to build (no 3D camera rig) and keeps the viewer's attention on the particle choreography rather than the camera path.

---

## Replication Priority

For AE ExtendScript replication, build these elements in order of impact:

| Priority | Element | Why | Difficulty |
|----------|---------|-----|-----------|
| 1 | Particle sphere + rotation | Hero visual, most memorable | Medium |
| 2 | Split composition | Strong layout pattern | Low |
| 3 | Subtitle system with line | Used throughout, high polish signal | Low |
| 4 | Generation visualization | Novel effect, high wow factor | Medium |
| 5 | Content mosaic | Good for showing range | Low |
| 6 | HUD infographic | Technical credibility | Medium |
| 7 | Embossed title cards | Opening impact | High (texture work) |
| 8 | UI card mockup | Product demo | High (many sub-elements) |
