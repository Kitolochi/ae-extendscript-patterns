# HUD & Technical Overlays

Data visualization overlays, engineering readouts, and measurement annotations layered on top of content. The Luma Uni-1 video uses this at t=35 with "SUSPENSION SYSTEM", "CAB", numerical readouts, and annotation lines.

## Data Label with Leader Line

A text label connected to a point on the frame by a thin line. Standard engineering annotation pattern.

```js
// The target point (where the line points to)
var targetX = 800; var targetY = 400;
// The label position (where the text sits)
var labelX = 1100; var labelY = 300;

// Leader line
var line = comp.layers.addShape();
line.name = "Leader Line";
var lc = line.property("Contents");
var lg = lc.addProperty("ADBE Vector Group");
var path = lg.property("Contents").addProperty("ADBE Vector Shape - Group");
var shape = new Shape();
// L-shaped leader: horizontal run, then vertical drop to target
var midX = labelX;
shape.vertices = [[labelX, labelY + 10], [midX, targetY], [targetX, targetY]];
shape.closed = false;
path.property("Path").setValue(shape);

var stroke = lg.property("Contents").addProperty("ADBE Vector Graphic - Stroke");
stroke.property("Color").setValue([0.4, 0.6, 1]);  // cool blue
stroke.property("Stroke Width").setValue(1);
stroke.property("Opacity").setValue(60);

// Endpoint dot
var dotGrp = lc.addProperty("ADBE Vector Group");
var dot = dotGrp.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
dot.property("Size").setValue([6, 6]);
var dotFill = dotGrp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
dotFill.property("Color").setValue([0.4, 0.6, 1]);
dotGrp.property("Transform").property("Position").setValue([targetX, targetY]);

// Label text
var label = comp.layers.addText("SUSPENSION SYSTEM");
var td = label.sourceText.value;
td.resetCharStyle();
td.fontSize = 14;
td.fillColor = [0.8, 0.9, 1];
td.font = "Consolas";  // monospace for technical feel
td.tracking = 200;     // wide tracking = technical typography
label.sourceText.setValue(td);
label.position.setValue([labelX, labelY]);
```

**Wide letter tracking** (200+ thousandths of an em) is the single strongest signal that text is "technical" rather than "readable." Combined with monospace font and small size (12-16px), it reads as instrument-panel typography.

## Animated Draw-On Line

The leader line draws itself from label to target, rather than appearing instantly.

```js
var trimPath = lg.property("Contents").addProperty("ADBE Vector Filter - Trim");
trimPath.property("End").expression = [
  'var f=timeToFrames(time);',
  'ease(f, 20, 40, 0, 100);'
].join('\\n');
```

Trim Paths `End` property animates from 0% to 100%, revealing the stroke progressively. The 20-frame duration (0.33s at 60fps) is fast enough to feel snappy, slow enough to be visible.

## Numerical Readout (Counting Up)

Numbers that count from zero to a target value, used for statistics and measurements.

```js
var numLayer = comp.layers.addText("0");
var td = numLayer.sourceText.value;
td.resetCharStyle();
td.fontSize = 32;
td.fillColor = [1, 1, 1];
td.font = "Consolas";
numLayer.sourceText.setValue(td);
numLayer.position.setValue([1100, 350]);

var targetValue = 27572;

numLayer.sourceText.expression = [
  'var f=timeToFrames(time);',
  'var n=Math.floor(ease(f, 25, 55, 0, ' + targetValue + '));',
  // Format with comma separators
  'var s=n.toString();',
  'var formatted="";',
  'for(var i=0;i<s.length;i++){',
  '  if(i>0&&(s.length-i)%3===0) formatted+=",";',
  '  formatted+=s.charAt(i);',
  '}',
  'formatted;'
].join('\\n');
```

The `ease` function makes the count accelerate in and decelerate out. Linear counting looks mechanical. For very large numbers (100,000+), use `linear` instead — the eased slow-start at the beginning makes small digits flicker too fast.

## Corner Brackets (Focus Frame)

Technical brackets at corners of an area, framing a region of interest.

```js
var bracketSize = 30;  // length of each bracket arm
var sw = 2;            // stroke width
var regions = [
  { x: 600, y: 300, w: 400, h: 300 }  // target area
];

for (var r = 0; r < regions.length; r++) {
  var reg = regions[r];
  var corners = [
    { x: reg.x, y: reg.y, dx: 1, dy: 1 },              // top-left
    { x: reg.x + reg.w, y: reg.y, dx: -1, dy: 1 },     // top-right
    { x: reg.x, y: reg.y + reg.h, dx: 1, dy: -1 },     // bottom-left
    { x: reg.x + reg.w, y: reg.y + reg.h, dx: -1, dy: -1 } // bottom-right
  ];

  var brackets = comp.layers.addShape();
  brackets.name = "Focus Brackets";
  var bc = brackets.property("Contents");

  for (var c = 0; c < 4; c++) {
    var cn = corners[c];
    var bg = bc.addProperty("ADBE Vector Group");
    var bp = bg.property("Contents").addProperty("ADBE Vector Shape - Group");
    var bs = new Shape();
    bs.vertices = [
      [cn.x + cn.dx * bracketSize, cn.y],
      [cn.x, cn.y],
      [cn.x, cn.y + cn.dy * bracketSize]
    ];
    bs.closed = false;
    bp.property("Path").setValue(bs);
    var bst = bg.property("Contents").addProperty("ADBE Vector Graphic - Stroke");
    bst.property("Color").setValue([0.4, 0.6, 1]);
    bst.property("Stroke Width").setValue(sw);
  }
}
```

Four L-shaped brackets at corners with no connecting lines between them. The empty space between brackets draws the eye into the framed region more than a full border would.

## Scan Line Effect

Horizontal line that sweeps vertically through the composition, suggesting an active scan or analysis.

```js
var scanLine = comp.layers.addShape();
scanLine.name = "Scan Line";
var sc = scanLine.property("Contents");
var sg = sc.addProperty("ADBE Vector Group");
var sr = sg.property("Contents").addProperty("ADBE Vector Shape - Rect");
sr.property("Size").setValue([1920, 2]);
var sf = sg.property("Contents").addProperty("ADBE Vector Graphic - Fill");
sf.property("Color").setValue([0.3, 0.7, 1]);  // cyan-blue
sf.property("Opacity").setValue(50);

scanLine.position.expression = [
  'var f=timeToFrames(time);',
  'var y=linear(f, 30, 90, -20, 1100);',  // top to bottom sweep
  '[960, y]'
].join('\\n');

scanLine.opacity.expression = [
  'var f=timeToFrames(time);',
  'var fadeIn=linear(f, 30, 35, 0, 100);',
  'var fadeOut=linear(f, 85, 90, 100, 0);',
  'Math.min(fadeIn, fadeOut);'
].join('\\n');

// Trailing glow behind the scan line
var glow = comp.layers.addShape();
glow.name = "Scan Glow";
var gc = glow.property("Contents");
var gg = gc.addProperty("ADBE Vector Group");
var gr = gg.property("Contents").addProperty("ADBE Vector Shape - Rect");
gr.property("Size").setValue([1920, 60]);
var gf = gg.property("Contents").addProperty("ADBE Vector Graphic - Fill");
gf.property("Color").setValue([0.2, 0.5, 1]);
gf.property("Opacity").setValue(15);

glow.position.expression = [
  'var f=timeToFrames(time);',
  'var y=linear(f, 30, 90, -20, 1100);',
  '[960, y-30]'  // trails behind scan line
].join('\\n');

var glowBlur = glow.property("Effects").addProperty("ADBE Gaussian Blur 2");
glowBlur.property("Blurriness").setValue(20);
```

The trailing glow (60px tall, 15% opacity, 20px blur) sells the scan line as emitting light. Without it, a 2px line by itself looks too thin and clinical.

## Ambient Grid

Faint grid lines across the entire frame, suggesting a coordinate system.

```js
// Horizontal and vertical lines at regular intervals
var gridLayer = comp.layers.addShape();
gridLayer.name = "HUD Grid";
var gc = gridLayer.property("Contents");

var spacing = 120;
// Vertical lines
for (var x = spacing; x < 1920; x += spacing) {
  var vg = gc.addProperty("ADBE Vector Group");
  var vp = vg.property("Contents").addProperty("ADBE Vector Shape - Group");
  var vs = new Shape();
  vs.vertices = [[x, 0], [x, 1080]];
  vs.closed = false;
  vp.property("Path").setValue(vs);
  var vst = vg.property("Contents").addProperty("ADBE Vector Graphic - Stroke");
  vst.property("Color").setValue([0.3, 0.5, 0.8]);
  vst.property("Stroke Width").setValue(0.5);
  vst.property("Opacity").setValue(8);  // barely visible
}

// Horizontal lines
for (var y = spacing; y < 1080; y += spacing) {
  var hg = gc.addProperty("ADBE Vector Group");
  var hp = hg.property("Contents").addProperty("ADBE Vector Shape - Group");
  var hs = new Shape();
  hs.vertices = [[0, y], [1920, y]];
  hs.closed = false;
  hp.property("Path").setValue(hs);
  var hst = hg.property("Contents").addProperty("ADBE Vector Graphic - Stroke");
  hst.property("Color").setValue([0.3, 0.5, 0.8]);
  hst.property("Stroke Width").setValue(0.5);
  hst.property("Opacity").setValue(8);
}
```

8% opacity at 0.5px stroke width creates a grid that's felt rather than seen. Above 15%, the grid competes with content. Below 5%, it vanishes on most displays.

**Performance note**: 1920/120 = 16 vertical + 1080/120 = 9 horizontal = 25 shape groups. Safe for a single send() call.
