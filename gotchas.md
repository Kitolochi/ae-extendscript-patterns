# AE ExtendScript Gotchas

Hard-won lessons from automated After Effects composition building via CEP bridge.

## Text Layers

### `new TextDocument()` is unusable
Creates an unassociated document object that can't be `setValue`'d on any layer.

```js
// WRONG — throws "Unable to set value as it is not associated with a layer"
var doc = new TextDocument("Hello");
layer.sourceText.setValue(doc);

// CORRECT — create text layer first, then modify its existing doc
var layer = comp.layers.addText("Hello");
var doc = layer.sourceText.value;
doc.fontSize = 36;
doc.fillColor = [1, 1, 1];
doc.font = "Arial-BoldMT";
layer.sourceText.setValue(doc);
```

## Effects

### Drop Shadow color is `"Shadow Color"`, not `"Color"`
```js
// WRONG — returns null, causes "null is not an object"
fx.property("Color").setValue([0, 0.5, 1]);

// CORRECT
fx.property("Shadow Color").setValue([0, 0.5, 1]);
```

### No duplicate effects
Can't add two instances of the same effect (e.g., two Drop Shadows). AE throws "Object is invalid". Use a different effect for the second purpose, or precomp the layer and apply the effect on the precomp.

### ADBE Glow (Glo2) fails on text layers
Text layers reject the Glow effect. Use Drop Shadow with `Distance=0` as a glow substitute:
```js
var glow = textLayer.property("Effects").addProperty("ADBE Drop Shadow");
glow.property("Shadow Color").setValue([0, 0.5, 1]);
glow.property("Opacity").setValue(200);
glow.property("Distance").setValue(0);
glow.property("Softness").setValue(40);
```

### ADBE Stroke renders on full layer bounds
The Stroke effect traces the entire layer boundary, not mask shapes. On a masked layer, this creates visible rectangles outside the mask. Use shape layer strokes or a dedicated border approach instead.

### Effect property ranges

| Effect | Property | Range | Notes |
|--------|----------|-------|-------|
| ADBE Stroke | Opacity | 0–1 | NOT 0–255. 0.15 = 15% |
| ADBE Ramp | Blend With Original | 0–1 | |
| ADBE Drop Shadow | Opacity | 0–255 | Standard AE range |
| ADBE Noise | Amount of Noise | 0–100 | Percentage |

### Transform Effect (ADBE Geometry2) requires matchNames
Display names don't work. Must use matchNames for properties:

| matchName | Property |
|-----------|----------|
| `ADBE Geometry2-0003` | Scale Height |
| `ADBE Geometry2-0004` | Scale Width |
| `ADBE Geometry2-0007` | Skew |
| `ADBE Geometry2-0008` | Skew Axis |

### ADBE Grid renders rectangular cells only
No hex or circular grid patterns available. Build custom grids with shape layer paths.

## Masks

### Mask API uses direct properties, not `.property()` strings
```js
// WRONG — causes null reference
mask.property("Mask Mode").setValue(MaskMode.SUBTRACT);
mask.property("Mask Feather").setValue([300, 300]);
mask.property("Mask Path").setValue(shapePath);

// CORRECT
mask.maskMode = MaskMode.SUBTRACT;
mask.maskFeather.setValue([300, 300]);
mask.property("ADBE Mask Shape").setValue(shapePath);
```

The mask path matchName is `"ADBE Mask Shape"`. The property name `"Mask Path"` works for `.addProperty()` on the Masks group (`"ADBE Mask Atom"`), but not for setValue on the mask itself.

### Bezier ellipse: use 4-point approximation
A 32-point trig loop causes null reference errors. Use the standard 4-point bezier circle with kappa = 0.5522847498:

```js
var cx = 960; var cy = 540; var rx = 1100; var ry = 700;
var k = 0.5522847498;
var ms = new Shape();
ms.vertices = [[cx, cy-ry], [cx+rx, cy], [cx, cy+ry], [cx-rx, cy]];
ms.inTangents = [[-rx*k, 0], [0, -ry*k], [rx*k, 0], [0, ry*k]];
ms.outTangents = [[rx*k, 0], [0, ry*k], [-rx*k, 0], [0, -ry*k]];
ms.closed = true;
```

### Rounded corners mask pattern
For a rectangle with radius `r` on a `w` x `h` solid:
```js
var r = 20; var w = 720; var h = 420;
var ms = new Shape();
ms.vertices = [[r,0],[w-r,0],[w,0],[w,r],[w,h-r],[w,h],[w-r,h],[r,h],[0,h],[0,h-r],[0,r],[0,0]];
ms.inTangents = [[0,0],[0,0],[r*0.55,0],[0,0],[0,0],[0,r*0.55],[0,0],[0,0],[-r*0.55,0],[0,0],[0,0],[0,-r*0.55]];
ms.outTangents = [[0,0],[r*0.55,0],[0,0],[0,0],[0,r*0.55],[0,0],[0,0],[-r*0.55,0],[0,0],[0,0],[0,-r*0.55],[0,0]];
ms.closed = true;
```

## Layers & Rendering

### Solid layers show rectangular edges through blur
Even with 200px Gaussian Blur, solid layers reveal their rectangular bounds. Use shape layer ellipses for anything that needs soft edges:

```js
// WRONG — rectangle visible through blur
var orb = comp.layers.addSolid([0, 0.3, 1], "Orb", 600, 600, 1, dur);
var blur = orb.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(150);

// CORRECT — smooth elliptical falloff
var orb = comp.layers.addShape();
orb.name = "Orb";
var grp = orb.property("Contents").addProperty("ADBE Vector Group");
var ell = grp.property("Contents").addProperty("ADBE Vector Shape - Ellipse");
ell.property("Size").setValue([600, 600]);
var fill = grp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
fill.property("Color").setValue([0, 0.3, 1]);
var blur = orb.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").setValue(150);
```

### Layer move API pitfalls
`layer.moveAfter(target)` and `layer.moveBefore(target)` throw "Can not move a layer before or after itself" if the layer IS the target. Always check:
```js
if (cardIdx && comp.layer(cardIdx) !== refl) {
  refl.moveAfter(comp.layer(cardIdx));
}
```

## 3D & Lighting

### Material Options darken content
Enabling "Accepts Lights" on a 3D precomp darkens all content to the Ambient percentage. Point lights at distance aren't strong enough to compensate. Use ADD-blend overlay layers for ambient lighting instead of AE's lighting system.

### Individual 3D layers misalign under rotation
Each 3D layer rotates around its own anchor point. If multiple layers form a card, they fly apart. Precomp them first, then make the single precomp 3D:
```js
comp.layers.precompose(layerIndices, "Card Precomp", true);
// true = moveAllAttributes, preserves transforms
var precomp = comp.layer("Card Precomp");
precomp.threeDLayer = true;
precomp.rotationY.setValue(-12);
precomp.rotationX.setValue(6);
```

## Performance

### Heavy shape operations need breathing room
Creating 96+ shape groups in a single `send()` can make AE temporarily unresponsive, hitting the 45-second bridge timeout. Add 1000ms `sleep()` after heavy steps:
```js
await send('Build 96 hex cells', hexCode);
await sleep(1000); // let AE digest
await send('Next step', nextCode);
```
