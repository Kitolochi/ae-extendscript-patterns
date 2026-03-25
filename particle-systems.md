# Particle Systems

Scattered dots, drifting elements, and constellation formations built from shape layer groups.

## Sparse Particle Field

Small circles scattered across frame at random positions. The base pattern for ambient background motion.

```js
var grp = comp.layers.addShape();
grp.name = "Particles";
var contents = grp.property("Contents");

var count = 40;
for (var i = 0; i < count; i++) {
  var ellGrp = contents.addProperty("ADBE Vector Group");
  ellGrp.name = "P" + i;
  var ell = ellGrp.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
  var size = 2 + Math.random() * 6;  // 2-8px diameter
  ell.property("Size").setValue([size, size]);
  var fill = ellGrp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
  fill.property("Color").setValue([0.2, 0.5, 1]);  // blue
  fill.property("Opacity").setValue(30 + Math.random() * 50);  // 30-80%

  var startX = Math.random() * 1920;
  var startY = Math.random() * 1080;
  var phase = Math.random() * 6.28;
  var speed = 0.3 + Math.random() * 0.7;

  ellGrp.property("Transform").property("Position").expression = [
    'var f=timeToFrames(time);',
    'var x=' + startX + '+Math.sin(f*0.008+' + phase + ')*20;',
    'var y=' + startY + '-f*' + speed + ';',
    'var wrapped=((y%1080)+1080)%1080;',
    '[x,wrapped]'
  ].join('\\n');
}
```

Key decisions:
- **2-8px diameter range** gives depth — smaller dots read as farther away
- **30-80% opacity range** reinforces the depth illusion
- **Slow sine drift on X** prevents mechanical straight-line movement
- **Y wrapping via modulo** recycles particles that drift off-screen

## Constellation Formation (Sphere)

Particles start scattered, then migrate toward positions on a sphere surface. The Luma Uni-1 launch video uses a 4-phase sequence: horizontal band → radial explosion → convergence to glowing orb → dots arrange on sphere surface.

### Reference Values (from Luma Uni-1, 60fps)

| Phase | Frames | Duration | What happens |
|-------|--------|----------|-------------|
| Band scatter | f0-f120 | 2s | Dots in a flat horizontal band, ~200px tall |
| Radial burst | f120-f180 | 1s | Expand to full frame, spiral structure, void at center |
| Convergence | f180-f300 | 2s | Collapse to nebula with bright additive core (~80px) |
| Sphere settle | f300-f420 | 2s | Dots arrange on sphere surface, start rotating |

### Sphere Measurements
- **Diameter**: 400px (40% of 1920px frame width)
- **Position**: frame center, slightly above vertical center (960, 500)
- **Particle count**: 400-600 on visible hemisphere
- **Particle sizes**: 2-6px, with Z-depth scaling (larger = closer to camera)
- **Color gradient**: cyan [0.2, 0.8, 0.9] on left hemisphere, magenta [0.85, 0.2, 0.6] on right
- **Rotation speed**: ~12-15 deg/sec (0.0035 rad/frame at 60fps)
- **Dot distribution**: follows latitude/longitude curves, NOT random surface scatter

### Implementation

```js
var count = 80;  // per send() call — use 5 calls for 400 total
var cx = 960; var cy = 500;
var radius = 200;  // 400px diameter

for (var i = 0; i < count; i++) {
  var sx = Math.random() * 1920;
  var sy = Math.random() * 1080;

  // Sphere surface position (spherical coordinates)
  var theta = Math.random() * Math.PI * 2;
  var phi = Math.acos(2 * Math.random() - 1);
  var tx = cx + radius * Math.sin(phi) * Math.cos(theta);
  var ty = cy + radius * Math.cos(phi);
  var tz = Math.sin(phi) * Math.sin(theta);  // -1 to 1 (depth)

  var dotSize = 2 + (tz + 1) * 2;  // 2-6px, larger = closer

  // Hemisphere color: lerp cyan→magenta based on X position
  var hemiT = (tx - (cx - radius)) / (2 * radius);  // 0=left, 1=right
  var r = 0.2 + hemiT * 0.65;   // 0.2 → 0.85
  var g = 0.8 - hemiT * 0.6;    // 0.8 → 0.2
  var b = 0.9 - hemiT * 0.3;    // 0.9 → 0.6

  var ellGrp = contents.addProperty("ADBE Vector Group");
  ellGrp.name = "P" + i;
  var ell = ellGrp.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
  ell.property("Size").setValue([dotSize, dotSize]);
  var fill = ellGrp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
  fill.property("Color").setValue([r, g, b]);
  // Back-hemisphere dots are dimmer
  fill.property("Opacity").setValue(tz > 0 ? 80 + tz * 20 : 30 + (tz + 1) * 50);

  ellGrp.property("Transform").property("Position").expression = [
    'var f=timeToFrames(time);',
    'var gatherStart=180; var gatherEnd=300;',
    'var t=clamp((f-gatherStart)/(gatherEnd-gatherStart),0,1);',
    't=t*t*(3-2*t);',  // smoothstep easing
    'var x=linear(t,0,1,' + sx + ',' + tx + ');',
    'var y=linear(t,0,1,' + sy + ',' + ty + ');',
    '[x,y]'
  ].join('\\n');

  ellGrp.property("Transform").property("Opacity").expression = [
    'var f=timeToFrames(time);',
    'linear(f,120,200,10,100)*linear(f,DUR-18,DUR,1,0);'
  ].join('\\n');
}
```

### Bright Core (Convergence Phase)

During convergence (f180-f300), add a large additive glow at the center that peaks then fades as dots settle into the sphere shape.

```js
var core = comp.layers.addShape();
core.name = "Convergence Core";
var cc = core.property("Contents");
var cg = cc.addProperty("ADBE Vector Group");
var ce = cg.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
ce.property("Size").setValue([80, 80]);
var cf = cg.property("Contents").addProperty("ADBE Vector Graphic - Fill");
cf.property("Color").setValue([0.9, 0.95, 1]);  // near-white with slight blue
core.position.setValue([cx, cy]);
core.blendingMode = BlendingMode.ADD;

// Blur makes it glow
var blur = core.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(60);

// Peaks during convergence, fades as sphere forms
core.opacity.expression = [
  'var f=timeToFrames(time);',
  'var ramp=ease(f, 180, 240, 0, 80);',     // fade in during convergence
  'var hold=ease(f, 300, 360, 80, 0);',      // fade out as sphere settles
  'Math.min(ramp, f<300 ? 80 : hold);'
].join('\\n');
```

Add 2-3 small colored accent dots (cyan, magenta) inside the core glow for visual interest during the convergence phase.

The sphere projection uses standard spherical coordinates (theta, phi) mapped to 2D. The Z component (depth) scales both dot size and opacity instead of using actual 3D layers.

**Smoothstep** (`t*t*(3-2*t)`) makes the convergence feel organic. Linear interpolation looks mechanical; smoothstep accelerates in and decelerates out.

## Slow Rotation (Post-Formation)

After particles settle into the sphere, add gentle rotation. The Luma video rotates at ~12-15 deg/sec — enough to confirm the 3D shape without becoming distracting.

```js
// Combined gather + rotation expression
var rotSpeed = 0.0035;  // radians per frame = ~12 deg/sec at 60fps
ellGrp.property("Transform").property("Position").expression = [
  'var f=timeToFrames(time);',
  'var gatherStart=180; var gatherEnd=300;',
  'var t=clamp((f-gatherStart)/(gatherEnd-gatherStart),0,1);',
  't=t*t*(3-2*t);',
  'var baseX=linear(t,0,1,' + sx + ',' + tx + ');',
  'var baseY=linear(t,0,1,' + sy + ',' + ty + ');',
  // Post-formation rotation
  'var postF=Math.max(f-300,0);',
  'var angle=postF*' + rotSpeed + ';',
  'var dx=baseX-' + cx + '; var dy=baseY-' + cy + ';',
  'var rx=dx*Math.cos(angle)-dy*Math.sin(angle)+' + cx + ';',
  'var ry=dx*Math.sin(angle)+dy*Math.cos(angle)+' + cy + ';',
  'var finalX=f>300?rx:baseX;',
  'var finalY=f>300?ry:baseY;',
  '[finalX,finalY]'
].join('\\n');
```

| Rotation speed | Deg/sec | Full rotation | Feel |
|----------------|---------|---------------|------|
| 0.002 rad/frame | 7 | ~51s | Glacial, barely perceptible |
| 0.0035 rad/frame | 12 | ~30s | Luma Uni-1 reference speed |
| 0.005 rad/frame | 17 | ~21s | Active, noticeable |
| 0.008 rad/frame | 27 | ~13s | Fast, draws attention |

## Sphere Dispersal

After holding the sphere, scatter particles back into a loose cloud. The Luma video transitions from tight sphere (t=12) to dispersed multicolor cloud (t=14) in ~2 seconds. The cloud particles are larger (5-15px vs 2-6px on the sphere) and occupy the right 2/3 of frame.

```js
// Phase 3: sphere → dispersed cloud
// Each particle gets a random "cloud" target position
var cloudX = 550 + Math.random() * 1250;  // right 2/3
var cloudY = 150 + Math.random() * 750;
var cloudSize = 5 + Math.random() * 10;    // 5-15px (larger than sphere dots)

ellGrp.property("Transform").property("Position").expression = [
  'var f=timeToFrames(time);',
  // ... gather phase (f180-f300) ...
  // ... rotation phase (f300-f420) ...
  // Dispersal phase (f420-f540)
  'var dStart=420; var dEnd=540;',
  'var d=clamp((f-dStart)/(dEnd-dStart),0,1);',
  'd=d*d*(3-2*d);',  // smoothstep
  'if(f>dStart){',
  '  var fx=linear(d,0,1,finalX,' + cloudX + ');',
  '  var fy=linear(d,0,1,finalY,' + cloudY + ');',
  '  [fx,fy];',
  '} else { [finalX,finalY]; }'
].join('\\n');

// Scale up during dispersal
ell.property("Size").expression = [
  'var f=timeToFrames(time);',
  'var s=linear(f, 420, 540, ' + dotSize + ', ' + cloudSize + ');',
  '[s,s]'
].join('\\n');
```

The size increase from 2-6px to 5-15px during dispersal gives a "camera zooming into the particles" feel. The cloud state holds with slow ambient drift (~0.5px/frame) until the scene transitions.

## Code Particles (Colored Rectangles)

Small colored rectangles drifting upward, simulating code fragments. Used behind UI cards to suggest programming activity.

```js
var colors = [
  [0.2, 0.5, 1],    // blue
  [0.16, 0.82, 0.56], // emerald
  [0.6, 0.4, 1],    // purple
  [1, 0.6, 0.2]     // amber
];

for (var i = 0; i < 12; i++) {
  var rectGrp = contents.addProperty("ADBE Vector Group");
  var rect = rectGrp.property("Contents").addProperty("ADBE Vector Shape - Rect");
  var w = 8 + Math.random() * 30;
  var h = 2 + Math.random() * 4;
  rect.property("Size").setValue([w, h]);

  var fill = rectGrp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
  var c = colors[Math.floor(Math.random() * colors.length)];
  fill.property("Color").setValue(c);
  fill.property("Opacity").setValue(15 + Math.random() * 25);

  var startX = 600 + Math.random() * 700;
  var startY = 300 + Math.random() * 500;
  var drift = 0.4 + Math.random() * 0.8;
  var phase = Math.random() * 6.28;

  rectGrp.property("Transform").property("Position").expression = [
    'var f=timeToFrames(time);',
    'var x=' + startX + '+Math.sin(f*0.02+' + phase + ')*15;',
    'var y=' + startY + '-f*' + drift + ';',
    'var wrapped=((y%600)+600)%600+200;',
    '[x,wrapped]'
  ].join('\\n');
}
```

Rectangles read as code tokens at a distance. Width variation (8-38px) mimics different token lengths. Height stays thin (2-6px) to maintain the "line of code" metaphor.

## Performance Notes

- **40 particles**: safe for a single shape layer, executes in one send() call
- **80+ particles**: split into two send() calls (40 each) with 500ms sleep between
- **120+ particles**: split into three calls with 1000ms sleep — AE becomes unresponsive during heavy shape group creation
- Each particle's expression evaluates per-frame, so keep math minimal. Avoid trig functions beyond one sin/cos per expression when particle count exceeds 60.
