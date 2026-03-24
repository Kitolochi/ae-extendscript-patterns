# Background Patterns

Techniques for building dark, atmospheric backgrounds in AE compositions.

## Radial Gradient Base

Never use a flat solid. Apply `ADBE Ramp` with radial shape for a lighter center that falls off to dark edges. Add scatter to prevent color banding.

```js
var bg = comp.layers.addSolid(bgColor, "BG", 1920, 1080, 1, comp.duration);
var ramp = bg.property("Effects").addProperty("ADBE Ramp");
ramp.property("Start of Ramp").setValue([960, 540]);     // center
ramp.property("Start Color").setValue([0.11, 0.11, 0.13]); // surface1
ramp.property("End of Ramp").setValue([960, 0]);
ramp.property("End Color").setValue([0.05, 0.05, 0.06]);   // bg
ramp.property("Ramp Shape").setValue(2);    // 2 = radial
ramp.property("Ramp Scatter").setValue(30); // break banding
```

## Color Washes

Large, soft shape layer ellipses in ADD blend mode. Position off-center for asymmetry. Standard palette: blue wash bottom-left, purple wash top-right.

```js
var wash = comp.layers.addShape();
wash.name = "BG Blue Wash";
var grp = wash.property("Contents").addProperty("ADBE Vector Group");
var ell = grp.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
ell.property("Size").setValue([1400, 1000]);
var fill = grp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
fill.property("Color").setValue([0.04, 0.06, 0.16]);
wash.position.setValue([400, 750]);       // bottom-left
wash.blendingMode = BlendingMode.ADD;
var blur = wash.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(300);
wash.opacity.setValue(60);
```

Key parameters:
- Ellipse size: 1200–1400px
- Blur: 300px (heavy — must be shape layer, not solid, to avoid rectangular edges)
- Opacity: 50–60%
- Blend: ADD

## Scan Lines

Diagonal line texture at 2.5% opacity. Reads as "digital" without demanding attention.

```js
var scan = comp.layers.addShape();
scan.name = "Scan Lines";
for (var i = 0; i < 20; i++) {
  var grp = scan.property("Contents").addProperty("ADBE Vector Group");
  var path = grp.property("Contents").addProperty("ADBE Vector Shape - Group");
  var shape = new Shape();
  shape.vertices = [[i * 120 - 200, 0], [i * 120 + 800, 1080]];
  shape.closed = false;
  path.property("Path").setValue(shape);
  var stroke = grp.property("Contents").addProperty("ADBE Vector Graphic - Stroke");
  stroke.property("Color").setValue([0, 0.47, 1]);
  stroke.property("Stroke Width").setValue(0.5);
  stroke.property("Opacity").setValue(40);
}
// Fade out during exit
scan.opacity.expression = 'var f=timeToFrames(time); linear(f, DUR-24, DUR-6, 2.5, 0);';
```

## Aurora

Soft horizontal glow near the bottom edge. Shape ellipse (not solid) to avoid rectangular artifacts.

```js
var aurora = comp.layers.addShape();
aurora.name = "Aurora";
var grp = aurora.property("Contents").addProperty("ADBE Vector Group");
var ell = grp.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
ell.property("Size").setValue([1200, 120]);  // wide and thin
var fill = grp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
fill.property("Color").setValue(purpleColor);
aurora.position.setValue([960, 1020]);       // near bottom edge
aurora.blendingMode = BlendingMode.SCREEN;
var blur = aurora.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(200);
// Gentle horizontal drift
aurora.position.expression = 'var f=timeToFrames(time); value+[Math.sin(f*0.012)*60,0]';
aurora.opacity.expression = 'var f=timeToFrames(time); linear(f,DUR-24,DUR-6,20,0);';
```

Key: push it far enough down (y=1020+) that it's an ambient glow at the screen edge, not a visible band.
