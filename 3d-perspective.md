# 3D Perspective & Isometric Cards

Techniques for making 3D product-showcase-style tilted UI cards in AE.

## The Precomp Approach

Individual 3D layers each rotate around their own anchor point. A card made of 30 layers will explode apart under rotation. The solution: precompose first, then make the single precomp layer 3D.

```js
// 1. Collect all card layer indices by name
var cardNames = [
  "Queue Card BG", "Card Glow", "Header Text",
  "Task1 BG", "Task1 Title", "Status Badge",
  // ... all layers that form the card
];
var indices = [];
for (var i = 1; i <= comp.numLayers; i++) {
  for (var j = 0; j < cardNames.length; j++) {
    if (comp.layer(i).name === cardNames[j]) {
      indices.push(i);
      break;
    }
  }
}

// 2. Precompose (true = moveAllAttributes, preserves transforms)
comp.layers.precompose(indices, "Card Precomp", true);

// 3. Find and configure the precomp layer
var precomp = null;
for (var i = 1; i <= comp.numLayers; i++) {
  if (comp.layer(i).name === "Card Precomp") { precomp = comp.layer(i); break; }
}
precomp.threeDLayer = true;
precomp.rotationY.expression = 'var f=timeToFrames(time); linear(f,0,280,-15,-10);';
precomp.rotationX.setValue(6);
```

## Rotation Values

For a "tech product showcase" isometric look:

| Property | Value | Notes |
|----------|-------|-------|
| rotationY | -15 → -10 (animated) | Left edge closer, right recedes. Slow drift adds life. |
| rotationX | 6 (static) | Slight top-back lean. Subtle but visible. |
| Camera Zoom | 900 | Lower = stronger perspective distortion |

Animate rotationY for gentle drift rather than a locked angle. 5 degrees of travel over the full shot duration is enough to feel alive.

## Camera Setup

AE needs a camera layer for 3D perspective to render.

```js
var cam = comp.layers.addCamera("Perspective Cam", [960, 540]);
cam.property("Camera Options").property("Zoom").setValue(900);
cam.position.setValue([960, 540, -1200]);
cam.pointOfInterest.setValue([960, 540, 0]);
```

Lower zoom = wider FOV = more dramatic perspective. 900–1200 is the useful range for product cards. Below 700, elements near the edges distort too much.

## Camera Control Null

A null layer for camera drift (scale, position) that parents all card elements. The AE camera handles perspective; the null handles framing.

```js
var camNull = comp.layers.addNull();
camNull.name = "Camera Control";
camNull.position.setValue([960, 540]);
camNull.anchorPoint.setValue([0, 0]);
// Slow zoom out: 103% → 100%
camNull.scale.expression = [
  'var f=timeToFrames(time);',
  'var s=linear(f,0,DUR,103,100);',
  '[s,s]'
].join('\\n');
// Slow drift: x 20 → -6
camNull.position.expression = [
  'var f=timeToFrames(time);',
  'var dx=linear(f,0,DUR,20,-6);',
  'value+[dx,0]'
].join('\\n');
```

Parent card layers to this null before precomposing. The precomp inherits the drift.

## Ground Reflection

Duplicate the card precomp, flip vertically, push down, blur, low opacity. Sells "floating above a reflective surface."

```js
// Find the precomp source in the project panel
var precompSrc = null;
for (var i = 1; i <= app.project.numItems; i++) {
  if (app.project.item(i) instanceof CompItem
      && app.project.item(i).name === "Card Precomp") {
    precompSrc = app.project.item(i);
    break;
  }
}

var refl = comp.layers.add(precompSrc);
refl.name = "Card Reflection";
refl.threeDLayer = true;
refl.rotationY.expression = cardRotationYExpression; // match card
refl.rotationX.setValue(-6);                          // inverted
refl.scale.setValue([100, -100, 100]);                // flip vertical
refl.position.setValue([960, 1060, 0]);               // below card
refl.blendingMode = BlendingMode.ADD;
refl.opacity.setValue(14);                            // very faint
var blur = refl.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(12);

// Move behind card in layer order
var cardIdx = null;
for (var i = 1; i <= comp.numLayers; i++) {
  if (comp.layer(i).name === "Card Precomp") { cardIdx = i; break; }
}
if (cardIdx) refl.moveAfter(comp.layer(cardIdx));
```

Key: keep opacity at 10-15%. Higher looks fake. The reflection should be barely perceptible.

## Why Not Material Options + Lights

AE's built-in lighting system (`Accepts Lights`, `Ambient`, `Diffuse`, `Specular`) darkens the entire 3D precomp to the Ambient percentage, then only brightens where lights hit. The result:

- Card content goes visibly darker
- Point lights at realistic distances don't compensate enough
- Text becomes hard to read

Use overlay effects instead:
- ADD-blend shape layer orbs for ambient glow
- Edge light strip (3D shape matching card rotation)
- Shimmer sweep for specular highlights
- Card glow layer behind for backlight separation
