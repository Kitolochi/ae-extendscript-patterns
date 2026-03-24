# Glass Card Construction

Building a glassmorphism UI card in After Effects via ExtendScript.

## Card Background

Solid layer with rounded corners mask, diagonal gradient, thin border stroke, and deep shadow.

```js
var card = comp.layers.addSolid(surfaceColor, "Queue Card BG", 720, 420, 1, comp.duration);
card.position.setValue([960, 540]);

// 1. Rounded corners mask
var mask = card.property("Masks").addProperty("ADBE Mask Atom");
var ms = new Shape();
var r = 20; var w = 720; var h = 420;
ms.vertices = [
  [r,0],[w-r,0],[w,0],[w,r],
  [w,h-r],[w,h],[w-r,h],[r,h],
  [0,h],[0,h-r],[0,r],[0,0]
];
ms.inTangents = [
  [0,0],[0,0],[r*0.55,0],[0,0],
  [0,0],[0,r*0.55],[0,0],[0,0],
  [-r*0.55,0],[0,0],[0,0],[0,-r*0.55]
];
ms.outTangents = [
  [0,0],[r*0.55,0],[0,0],[0,0],
  [0,r*0.55],[0,0],[0,0],[-r*0.55,0],
  [0,0],[0,0],[0,-r*0.55],[0,0]
];
ms.closed = true;
mask.property("Mask Path").setValue(ms);

// 2. Diagonal gradient
var grad = card.property("Effects").addProperty("ADBE Ramp");
grad.property("Start of Ramp").setValue([0, 0]);
grad.property("End of Ramp").setValue([720, 420]);
grad.property("Start Color").setValue([0.102, 0.102, 0.122]);
grad.property("End Color").setValue([0.075, 0.075, 0.086]);
grad.property("Ramp Shape").setValue(1); // linear

// 3. Border stroke — white at 15% opacity
var border = card.property("Effects").addProperty("ADBE Stroke");
border.property("Color").setValue([1, 1, 1]);
border.property("Brush Size").setValue(1);
border.property("Opacity").setValue(0.15); // 0-1 range, NOT 0-255

// 4. Drop shadow
var shadow = card.property("Effects").addProperty("ADBE Drop Shadow");
shadow.property("Opacity").setValue(180);
shadow.property("Direction").setValue(180);
shadow.property("Distance").setValue(40);
shadow.property("Softness").setValue(80);
shadow.property("Shadow Color").setValue([0, 0, 0]);
```

## Card Glow (Behind)

Elliptical shape layer, not solid, to avoid rectangular artifacts through blur.

```js
var glow = comp.layers.addShape();
glow.name = "Card Glow";
var grp = glow.property("Contents").addProperty("ADBE Vector Group");
var ell = grp.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
ell.property("Size").setValue([700, 400]);
var fill = grp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
fill.property("Color").setValue(blueColor);
glow.position.setValue([960, 540]);
glow.blendingMode = BlendingMode.ADD;
var blur = glow.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(80);
glow.opacity.expression = [
  'var f=timeToFrames(time);',
  'var fadeIn=ease(f,4,22,0,20);',
  'var pulse=1+0.15*Math.sin(f*0.03);',
  'var fadeOut=linear(f,DUR-18,DUR,1,0);',
  'fadeIn*pulse*fadeOut;'
].join('\\n');
```

## Glass Highlights

### Top-edge specular
Thin white solid at the card's top edge, slight blur. Sells the glass material.

```js
var hl = precomp.layers.addSolid([1,1,1], "Card Top Highlight", 700, 3, 1, dur);
hl.position.setValue([960, cardTopY]);
var blur = hl.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(2);
hl.opacity.setValue(35);
```

### Inner shadow at bottom
Black solid with feathered mask. Creates depth inside the card.

```js
var ish = precomp.layers.addSolid([0,0,0], "Card Inner Shadow", 680, 30, 1, dur);
ish.position.setValue([960, cardBottomY]);
var mask = ish.property("Masks").addProperty("ADBE Mask Atom");
var ms = new Shape();
ms.vertices = [[0,0],[680,0],[680,30],[0,30]];
ms.closed = true;
mask.property("Mask Path").setValue(ms);
mask.property("Mask Feather").setValue([0, 20]); // vertical feather only
ish.opacity.setValue(25);
```

### Shimmer sweep
One-time diagonal light sweep across the glass surface.

```js
var shimmer = precomp.layers.addSolid([1,1,1], "Shimmer Sweep", 120, 520, 1, dur);
shimmer.rotation.setValue(-20);
shimmer.blendingMode = BlendingMode.ADD;
var blur = shimmer.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(30);
// Sweep left→right between f50-f90
shimmer.position.expression = [
  'var f=timeToFrames(time);',
  'var sweepX=ease(f,50,90,400,1520);',
  '[sweepX, 540]'
].join('\\n');
shimmer.opacity.expression = [
  'var f=timeToFrames(time);',
  'var fadeIn=linear(f,50,60,0,8);',
  'var fadeOut=linear(f,80,90,8,0);',
  'Math.min(fadeIn,fadeOut);'
].join('\\n');
```

Place above the card BG but below card content in the precomp.

## Edge Lighting

Thin rectangle on the card's leading edge (left side for right-leaning tilt). Must be a 3D layer matching the card's rotation.

```js
var edge = comp.layers.addShape();
edge.name = "Card Edge Light";
edge.threeDLayer = true;
var grp = edge.property("Contents").addProperty("ADBE Vector Group");
var rect = grp.property("Contents").addProperty("ADBE Vector Shape - Rect");
rect.property("Size").setValue([40, 420]);
var fill = grp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
fill.property("Color").setValue(blueColor);
edge.position.setValue([620, 540, 0]); // card left edge + 20px inward
edge.rotationY.expression = cardRotationY; // match card tilt
edge.rotationX.setValue(6);             // match card tilt
edge.blendingMode = BlendingMode.ADD;
var blur = edge.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(20);
edge.opacity.setValue(50);
```

## Progress Bar

Animated fill using scale X from a left-anchored position.

```js
var bar = precomp.layers.addShape();
bar.name = "Progress Bar";
var grp = bar.property("Contents").addProperty("ADBE Vector Group");
var rect = grp.property("Contents").addProperty("ADBE Vector Shape - Rect");
rect.property("Size").setValue([620, 3]);
rect.property("Roundness").setValue(2);
var fill = grp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
fill.property("Color").setValue(emeraldColor);
bar.position.setValue([960, progressY]);
// Shift anchor point to left edge so scaleX grows rightward
bar.anchorPoint.setValue([-310, 0]); // half of 620
bar.scale.expression = [
  'var f=timeToFrames(time);',
  'var pct=linear(f,70,140,0,100);',
  '[pct,100]'
].join('\\n');
bar.opacity.setValue(60);
```
