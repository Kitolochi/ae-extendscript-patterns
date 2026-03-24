# Atmospheric Elements

Ambient motion and depth elements that fill the space around focal content.

## Soft Glow Orbs

Shape layer ellipses with heavy blur. ADD blend. Must use shape layers — solids show rectangular edges through blur.

```js
var orb = comp.layers.addShape();
orb.name = "Blue Depth Orb";
var grp = orb.property("Contents").addProperty("ADBE Vector Group");
var ell = grp.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
ell.property("Size").setValue([500, 500]);
var fill = grp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
fill.property("Color").setValue(blueColor);
orb.position.setValue([720, 420]);
orb.blendingMode = BlendingMode.ADD;
var blur = orb.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(150);
orb.opacity.expression = [
  'var f=timeToFrames(time);',
  'var fadeIn=ease(f,8,24,0,12);',
  'var pulse=0.8+0.2*Math.sin(f*0.025);',   // slow breathe
  'var fadeOut=linear(f,DUR-18,DUR,1,0);',
  'fadeIn*pulse*fadeOut;'
].join('\\n');
```

Typical values:
- Size: 400–500px
- Blur: 130–150px
- Opacity: 10–12% peak
- Pulse frequency: 0.025–0.03
- Use 2 orbs with different colors (blue upper-left, purple lower-right) for color contrast

## Concentric Rings

Expanding radar-pulse rings. 3 shape layer ellipses with staggered phase.

```js
for (var r = 0; r < 3; r++) {
  var ring = comp.layers.addShape();
  ring.name = "Ring " + r;
  var grp = ring.property("Contents").addProperty("ADBE Vector Group");
  var ell = grp.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
  ell.property("Size").setValue([400, 400]);
  var stroke = grp.property("Contents").addProperty("ADBE Vector Graphic - Stroke");
  stroke.property("Color").setValue(blueColor);
  stroke.property("Stroke Width").setValue(1);
  stroke.property("Opacity").setValue(30);
  ring.position.setValue([960, 540]);

  var offset = r * 60;  // stagger: 0, 60, 120 frames
  ring.scale.expression = [
    'var f=timeToFrames(time);',
    'var ringFrame=(f+' + offset + ')%180;',
    'var s=linear(ringFrame,0,180,30,250);',
    '[s,s]'
  ].join('\\n');
  ring.opacity.expression = [
    'var f=timeToFrames(time);',
    'var ringFrame=(f+' + offset + ')%180;',
    'var fadeIn=linear(ringFrame,0,30,15,12);',
    'var fadeOut=linear(ringFrame,120,180,1,0);',
    'var bgFade=linear(f,DUR-24,DUR-6,1,0);',
    'fadeIn*fadeOut*bgFade;'
  ].join('\\n');
}
```

Each ring cycles 30%→250% scale over 180 frames, fading in then out per cycle. The 60-frame offset creates a continuous pulse pattern.

## Floating Particles

18 tiny ellipses in a single shape layer, cycling through 3 colors.

```js
var parts = comp.layers.addShape();
parts.name = "Ambient Particles";
for (var i = 0; i < 18; i++) {
  var grp = parts.property("Contents").addProperty("ADBE Vector Group");
  var ell = grp.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
  var sz = 1.5 + (i % 3) * 1;
  ell.property("Size").setValue([sz, sz]);
  var fill = grp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
  var colors = [blueColor, emeraldColor, purpleColor];
  fill.property("Color").setValue(colors[i % 3]);

  var startX = (i * 113.7) % 1920;
  var startY = 60 + (i * 67.3) % 960;
  var speed = 0.1 + (i % 5) * 0.05;
  var phase = i * 2.1;

  grp.property("Transform").property("Position").expression = [
    'var f=timeToFrames(time);',
    'var x=' + startX + '+Math.cos(f*' + (speed*0.03) + '+' + phase + ')*18-960;',
    'var y=' + startY + '+Math.sin(f*' + (speed*0.05) + '+' + phase + ')*25-540;',
    '[x,y]'
  ].join('\\n');

  grp.property("Transform").property("Opacity").expression = [
    'var f=timeToFrames(time);',
    'var fadeIn=linear(f,' + (6+i*2) + ',' + (18+i*2) + ',0,45);',
    'var pulse=0.4+0.6*Math.sin(f*0.04+' + phase + ');',
    'var bgFade=linear(f,DUR-24,DUR-6,1,0);',
    'fadeIn*pulse*bgFade;'
  ].join('\\n');
}
```

Key design decisions:
- **Irrational multipliers** (113.7, 67.3, 2.1) distribute particles with no visible pattern
- **Per-particle phase** prevents synchronized motion
- **Size 1.5–3.5px** — visible but never distracting
- **45% peak opacity** with pulse — dim enough to stay ambient

## Code Particles

Small colored rectangles drifting upward behind the card. Represents "code being generated" without needing actual text (text in shape groups isn't possible).

```js
var codeParticles = comp.layers.addShape();
codeParticles.name = "Code Particles";
for (var i = 0; i < 12; i++) {
  var grp = codeParticles.property("Contents").addProperty("ADBE Vector Group");
  var rect = grp.property("Contents").addProperty("ADBE Vector Shape - Rect");
  var w = 8 + (i % 4) * 6;  // 8, 14, 20, 26px wide
  rect.property("Size").setValue([w, 2]);
  rect.property("Roundness").setValue(1);
  var fill = grp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
  fill.property("Color").setValue(colors[i % 3]);

  var startX = 700 + (i * 47) % 520;
  var startY = 380 + (i * 31) % 200;
  var speed = 0.3 + (i % 5) * 0.15;

  grp.property("Transform").property("Position").expression = [
    'var f=timeToFrames(time);',
    'var x=' + startX + '+Math.sin(f*0.015+' + (i*1.7) + ')*30-960;',
    'var y=' + startY + '-f*' + speed + '-540;',
    'var wrapped=((y+540)%600)-540+' + startY + '-540;',
    '[x,wrapped]'
  ].join('\\n');
}
```

The Y-axis wrapping (`%600`) creates continuous upward drift that recycles without visible popping.

## Data Flow Lines

Dashed, animated lines converging toward the center from screen edges.

```js
var flows = comp.layers.addShape();
flows.name = "Data Flow Lines";
var lineData = [
  {x1:-40, y1:320, x2:480, y2:420, delay:10},
  {x1:-40, y1:680, x2:420, y2:560, delay:18},
  {x1:1960, y1:280, x2:1200, y2:400, delay:14},
  // ... 6 total, from all edges
];
for (var i = 0; i < lineData.length; i++) {
  var d = lineData[i];
  var grp = flows.property("Contents").addProperty("ADBE Vector Group");
  var path = grp.property("Contents").addProperty("ADBE Vector Shape - Group");
  var shape = new Shape();
  shape.vertices = [[d.x1-960, d.y1-540], [d.x2-960, d.y2-540]];
  shape.closed = false;
  path.property("Path").setValue(shape);
  var stroke = grp.property("Contents").addProperty("ADBE Vector Graphic - Stroke");
  stroke.property("Color").setValue(blueColor);
  stroke.property("Stroke Width").setValue(0.8);
  stroke.property("Opacity").setValue(40);
  // Dashes
  stroke.property("Dashes").addProperty("ADBE Vector Stroke Dash 1").setValue(8);
  stroke.property("Dashes").addProperty("ADBE Vector Stroke Gap 1").setValue(16);
  // Animated dash offset — creates flow motion
  var dashOff = stroke.property("Dashes").addProperty("ADBE Vector Stroke Offset");
  dashOff.expression = 'timeToFrames(time) * -2.5';
}
```

The dash offset expression creates the illusion of data flowing along each line toward the center.
