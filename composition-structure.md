# Composition Structure

Patterns for organizing visual elements across the frame and structuring multi-shot narratives.

## Bookend Structure

Opening and closing shots share the same visual motif, creating narrative closure. The Luma Uni-1 video opens with a dark particle field ("They built intelligence in pieces") and closes with the same field ("This is UNI-1").

Implementation: build the bookend as a single precomp, use it twice.

```js
// Create the particle field precomp once
var fieldComp = app.project.items.addComp("Particle Field", 1920, 1080, 1, 10, 60);
// ... populate with particles (see particle-systems.md) ...

// Opening: use particle field, trim to first 5 seconds
var mainComp = __findComp("Main");
var opening = mainComp.layers.add(fieldComp);
opening.name = "Opening Field";
opening.inPoint = 0;
opening.outPoint = 5;

// Closing: same source, placed at end
var closing = mainComp.layers.add(fieldComp);
closing.name = "Closing Field";
closing.inPoint = 55;  // last 5 seconds
closing.outPoint = 60;

// Closing fades to black
closing.opacity.expression = [
  'var f=timeToFrames(time-inPoint);',
  'var dur=timeToFrames(outPoint-inPoint);',
  'linear(f, dur-30, dur, 100, 0);'
].join('\\n');
```

The closing version fades slower than the opening appears. Entrances feel best at 12-18 frames; exits work better at 24-30 frames. Asymmetric pacing prevents the bookend from feeling mechanical.

### Luma Uni-1 Bookend Timeline

The particle field serves as the bookend motif. Opening (t=3-9) shows particles scattered then converging. Closing (t=55-57) shows the same particles in a relaxed scatter. Between them: 46 seconds of content showcasing the product.

| Section | Time | Duration |
|---------|------|----------|
| Opening particle field | 0:03-0:09 | 6s |
| Sphere formation + hero shot | 0:09-0:16 | 7s |
| Product demos (UI, generation, content) | 0:17-0:52 | 35s |
| Content mosaic | 0:53-0:55 | 2s |
| Closing particle field | 0:55-0:57 | 2s |
| Final title + logo | 0:57-1:00 | 3s |

The opening gets more time (6s) because it establishes the visual language. The closing is shorter (2s) — the audience already recognizes the motif.

## Split Composition (Text + Visual)

Frame divided into two zones: typography on one side, visual element on the other. The Luma video at t=15 places "UNI-1" on the left third and the particle sphere on the right two-thirds.

```js
// Left zone: typography
var title = comp.layers.addText("UNI-1");
var td = title.sourceText.value;
td.resetCharStyle();
td.fontSize = 160;
td.fillColor = [1, 1, 1];
td.font = "Helvetica Neue";
td.tracking = 100;
title.sourceText.setValue(td);
title.position.setValue([380, 540]);  // left third center

// Right zone: visual element positioned in right 2/3
// (sphere constellation, product mockup, video playback, etc.)
var visual = mainComp.layers.add(visualPrecomp);
visual.position.setValue([1200, 540]);  // right-biased center
```

**Safe zones for split compositions:**
| Layout | Left zone | Right zone |
|--------|-----------|------------|
| 1/6 + 5/6 | x: 100-380 | x: 550-1820 |
| 1/3 + 2/3 | x: 100-580 | x: 660-1820 |
| 1/2 + 1/2 | x: 100-860 | x: 1060-1820 |
| 2/3 + 1/3 | x: 100-1260 | x: 1340-1820 |

The Luma Uni-1 split at t=14-16 uses a narrow 1/6 left zone for "UNI-1" text (x≈320, ~160px font) with the particle cloud filling the remaining 5/6. The text sits at vertical center (y=540).

The 80px gap between zones (the "gutter") prevents elements from feeling crowded. Title-safe margin: 100px from all edges.

## Content Mosaic Grid

Multiple images or video clips arranged in a tiled grid. The Luma video at t=42 shows manga/comic artwork in a collage layout.

```js
var tileData = [
  { x: 280, y: 300, w: 440, h: 480 },   // large left tile
  { x: 740, y: 300, w: 220, h: 230 },   // top-center
  { x: 740, y: 550, w: 220, h: 230 },   // bottom-center
  { x: 980, y: 300, w: 440, h: 480 },   // large right tile
  { x: 1440, y: 300, w: 220, h: 230 },  // far-right top
  { x: 1440, y: 550, w: 220, h: 230 },  // far-right bottom
];

for (var i = 0; i < tileData.length; i++) {
  var t = tileData[i];

  // Tile background (placeholder or imported footage)
  var tile = comp.layers.addShape();
  tile.name = "Tile " + (i + 1);
  var tc = tile.property("Contents");
  var tg = tc.addProperty("ADBE Vector Group");
  var tr = tg.property("Contents").addProperty("ADBE Vector Shape - Rect");
  tr.property("Size").setValue([t.w, t.h]);
  tr.property("Roundness").setValue(6);
  var tf = tg.property("Contents").addProperty("ADBE Vector Graphic - Fill");
  tf.property("Color").setValue([0.15 + Math.random() * 0.1, 0.12, 0.18]);

  tile.position.setValue([t.x, t.y]);

  // Staggered entrance
  var delay = i * 4;  // 4 frames between each tile
  tile.opacity.expression = [
    'var f=timeToFrames(time);',
    'ease(f, ' + delay + ', ' + (delay + 12) + ', 0, 100);'
  ].join('\\n');

  tile.scale.expression = [
    'var f=timeToFrames(time);',
    'var s=ease(f, ' + delay + ', ' + (delay + 12) + ', 90, 100);',
    '[s,s]'
  ].join('\\n');
}
```

**Stagger timing**: 4 frames between tiles gives a smooth cascade. 2 frames is too fast to perceive individual entrances. 8+ frames makes the grid feel sluggish. For 6 tiles at 4-frame stagger, the full cascade takes 36 frames (0.6s).

Tile sizes should follow a clear hierarchy: 1-2 large tiles, 2-4 medium tiles. A uniform grid reads as a gallery; mixed sizes read as editorial layout.

### Luma Uni-1 Mosaic Reference (t=53)

The content mosaic at t=53 uses an irregular grid of 12-15 image tiles:

| Property | Value |
|----------|-------|
| Large tiles | ~300×250px (2-3 of these) |
| Small tiles | ~120×120px (8-10 of these) |
| Gap | 8-12px between tiles |
| Tile rotation | ±2-3 degrees (subtle, not all tiles) |
| Parallax drift | ~0.3px/frame (barely perceptible float) |
| Background | pure black |
| Duration | 2 seconds before dissolving to closing particle field |

The images show mixed styles (portraits, illustrations, manga, 3D renders, photography) to demonstrate range. Some tiles overlap slightly for a casual, organic feel rather than rigid alignment.

## Subtitle Bar

Lower-third text with background bar for readability over varying content.

```js
var barY = 920;  // lower fifth of frame

// Semi-transparent bar
var bar = comp.layers.addShape();
bar.name = "Subtitle Bar";
var bc = bar.property("Contents");
var bg = bc.addProperty("ADBE Vector Group");
var br = bg.property("Contents").addProperty("ADBE Vector Shape - Rect");
br.property("Size").setValue([1920, 60]);
var bf = bg.property("Contents").addProperty("ADBE Vector Graphic - Fill");
bf.property("Color").setValue([0, 0, 0]);
bf.property("Opacity").setValue(40);
bar.position.setValue([960, barY]);

// Subtitle text
var sub = comp.layers.addText("This is the era of Unified Intelligence.");
var td = sub.sourceText.value;
td.resetCharStyle();
td.fontSize = 22;
td.fillColor = [1, 1, 1];
td.font = "Helvetica Neue";
td.tracking = 50;
td.justification = ParagraphJustification.CENTER_JUSTIFY;
sub.sourceText.setValue(td);
sub.position.setValue([960, barY + 2]);  // vertically centered in bar

// Timed appearance
sub.opacity.expression = [
  'var f=timeToFrames(time);',
  'var enter=ease(f, 60, 72, 0, 100);',
  'var exit=ease(f, 200, 212, 100, 0);',
  'Math.min(enter, exit);'
].join('\\n');
```

40% black bar opacity is the minimum for white text readability over bright content. 60% for maximum readability. Above 70%, the bar calls too much attention to itself.

## Scene Transitions

### Cross-dissolve via Opacity

```js
// Scene A fades out
sceneA.opacity.expression = [
  'var f=timeToFrames(time);',
  'linear(f, transitionStart, transitionStart+18, 100, 0);'
].join('\\n');

// Scene B fades in (starts slightly before A finishes)
sceneB.opacity.expression = [
  'var f=timeToFrames(time);',
  'linear(f, transitionStart-4, transitionStart+14, 0, 100);'
].join('\\n');
```

The 4-frame overlap (Scene B starts fading in before Scene A finishes) creates a true dissolve rather than a dip-to-black. Without overlap, you get a brief dark frame in the middle.

### Scale + Blur Push

The current scene scales up slightly and blurs, suggesting forward motion into the next scene.

```js
// Exit: scale 100→106%, blur 0→30
var exitScale = comp.layers.addSolid(bgColor, "Exit Push", 1920, 1080, 1, comp.duration);
exitScale.adjustmentLayer = true;
var fx = exitScale.property("Effects").addProperty("ADBE Geometry2");
fx.property("ADBE Geometry2-0003").expression =
  'var f=timeToFrames(time); linear(f, DUR-16, DUR, 100, 106);';
fx.property("ADBE Geometry2-0004").expression =
  'var f=timeToFrames(time); linear(f, DUR-16, DUR, 100, 106);';
exitScale.inPoint = secondsAtDurMinus16;

var exitBlur = comp.layers.addSolid(bgColor, "Exit Blur", 1920, 1080, 1, comp.duration);
exitBlur.adjustmentLayer = true;
var blur = exitBlur.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").expression =
  'var f=timeToFrames(time); linear(f, DUR-16, DUR, 0, 30);';
exitBlur.inPoint = secondsAtDurMinus16;
```

See [post-effects.md](post-effects.md) for full entry/exit transition patterns.
