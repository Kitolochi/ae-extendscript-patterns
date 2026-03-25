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

Particles start scattered, then migrate toward positions on a sphere surface. Used in the Luma Uni-1 launch video for the logo-to-sphere transition.

```js
// Phase 1: random start positions
// Phase 2: lerp toward sphere surface positions
var count = 80;
var cx = 1200; var cy = 540;  // sphere center (right third of frame)
var radius = 250;

for (var i = 0; i < count; i++) {
  // Random starting position
  var sx = Math.random() * 1920;
  var sy = Math.random() * 1080;

  // Target position on sphere surface (2D projection)
  var theta = Math.random() * Math.PI * 2;
  var phi = Math.acos(2 * Math.random() - 1);
  var tx = cx + radius * Math.sin(phi) * Math.cos(theta);
  var ty = cy + radius * Math.cos(phi);
  // Z controls dot size (depth simulation)
  var tz = Math.sin(phi) * Math.sin(theta);  // -1 to 1

  var dotSize = 3 + (tz + 1) * 2.5;  // 3-8px, larger = closer

  // ... create shape group with size=dotSize ...

  ellGrp.property("Transform").property("Position").expression = [
    'var f=timeToFrames(time);',
    'var gatherStart=60; var gatherEnd=120;',
    'var t=clamp((f-gatherStart)/(gatherEnd-gatherStart),0,1);',
    't=t*t*(3-2*t);',  // smoothstep
    'var x=linear(t,0,1,' + sx + ',' + tx + ');',
    'var y=linear(t,0,1,' + sy + ',' + ty + ');',
    '[x,y]'
  ].join('\\n');

  // Opacity: invisible → visible as they converge
  ellGrp.property("Transform").property("Opacity").expression = [
    'var f=timeToFrames(time);',
    'linear(f,40,80,10,100)*linear(f,DUR-18,DUR,1,0);'
  ].join('\\n');
}
```

The sphere projection uses standard spherical coordinates (theta, phi) mapped to 2D. The Z component (depth) scales dot size instead of using actual 3D layers — avoids the complexity of cameras and 3D layer management.

**Smoothstep** (`t*t*(3-2*t)`) makes the convergence feel organic. Linear interpolation looks mechanical; smoothstep accelerates in and decelerates out.

## Slow Rotation (Post-Formation)

After particles settle into the sphere, add gentle rotation:

```js
// After gather completes, rotate all positions around center
var rotSpeed = 0.003;  // radians per frame
ellGrp.property("Transform").property("Position").expression = [
  'var f=timeToFrames(time);',
  'var gatherStart=60; var gatherEnd=120;',
  'var t=clamp((f-gatherStart)/(gatherEnd-gatherStart),0,1);',
  't=t*t*(3-2*t);',
  'var baseX=linear(t,0,1,' + sx + ',' + tx + ');',
  'var baseY=linear(t,0,1,' + sy + ',' + ty + ');',
  // Post-formation rotation
  'var postF=Math.max(f-120,0);',
  'var angle=postF*' + rotSpeed + ';',
  'var dx=baseX-' + cx + '; var dy=baseY-' + cy + ';',
  'var rx=dx*Math.cos(angle)-dy*Math.sin(angle)+' + cx + ';',
  'var ry=dx*Math.sin(angle)+dy*Math.cos(angle)+' + cy + ';',
  'var finalX=f>120?rx:baseX;',
  'var finalY=f>120?ry:baseY;',
  '[finalX,finalY]'
].join('\\n');
```

At 0.003 rad/frame (60fps), the sphere completes a full rotation in ~35 seconds. For a 5-second hold, that's about 50 degrees of visible rotation — enough to confirm the shape is spherical without becoming distracting.

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
