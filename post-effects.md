# Post Effects & Polish

Global composition effects applied after all content layers.

## Vignette

Black solid with a subtracted, heavily feathered elliptical mask. Darkens corners without affecting the center.

```js
var vig = comp.layers.addSolid([0,0,0], "Vignette", 1920, 1080, 1, comp.duration);
vig.moveToBeginning(); // top of layer stack

var mask = vig.property("Masks").addProperty("ADBE Mask Atom");
var ms = new Shape();
var cx = 960; var cy = 540;
var rx = 1100; var ry = 700;  // slightly larger than frame
var k = 0.5522847498;         // bezier circle kappa

ms.vertices = [[cx, cy-ry], [cx+rx, cy], [cx, cy+ry], [cx-rx, cy]];
ms.inTangents = [[-rx*k, 0], [0, -ry*k], [rx*k, 0], [0, ry*k]];
ms.outTangents = [[rx*k, 0], [0, ry*k], [-rx*k, 0], [0, -ry*k]];
ms.closed = true;

mask.property("ADBE Mask Shape").setValue(ms);
mask.maskMode = MaskMode.SUBTRACT;   // cut out the ellipse center
mask.maskFeather.setValue([300, 300]); // heavy feather = soft falloff
vig.opacity.setValue(60);
```

The kappa constant (0.5522847498) creates a mathematically correct circular arc from 4 bezier points. Don't use a trig loop with 32+ points — it causes null reference errors in ExtendScript.

## Film Grain

Gray solid in OVERLAY blend mode with animated noise.

```js
var grain = comp.layers.addSolid([0.5, 0.5, 0.5], "Film Grain", 1920, 1080, 1, comp.duration);
grain.moveToBeginning();
grain.blendingMode = BlendingMode.OVERLAY;
grain.opacity.setValue(6);  // subtle — 4-8% range

var noise = grain.property("Effects").addProperty("ADBE Noise");
noise.property("Amount of Noise").setValue(50);
noise.property("Amount of Noise").expression = 'wiggle(60, 10)';
```

OVERLAY on a 50% gray solid is neutral — it only shows through the noise pattern. At 6% opacity, the grain is felt more than seen. Breaks up the CG-clean look.

## Entry Transition

Two adjustment layers: blur 24→0 and scale 105%→100% over the first 12 frames.

```js
// Entry blur
var entry = comp.layers.addSolid(bgColor, "Entry Blur", 1920, 1080, 1, comp.duration);
entry.adjustmentLayer = true;
entry.moveToBeginning();
var blur = entry.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").expression = 'var f=timeToFrames(time); linear(f,0,12,24,0);';
entry.outPoint = secondsAtFrame14; // trim to 14 frames

// Entry scale (Transform effect, not layer scale)
var entryS = comp.layers.addSolid(bgColor, "Entry Scale", 1920, 1080, 1, comp.duration);
entryS.adjustmentLayer = true;
entryS.moveToBeginning();
var fx = entryS.property("Effects").addProperty("ADBE Geometry2");
fx.property("ADBE Geometry2-0003").expression = 'var f=timeToFrames(time); linear(f,0,12,105,100);';
fx.property("ADBE Geometry2-0004").expression = 'var f=timeToFrames(time); linear(f,0,12,105,100);';
entryS.outPoint = secondsAtFrame14;
```

The Transform effect (ADBE Geometry2) scales the entire frame, not individual layers. Must use matchNames for Scale Height/Width (`-0003`, `-0004`).

## Exit Transition

Mirror of entry, plus a black solid fade to black.

```js
// Exit blur
var exitB = comp.layers.addSolid(bgColor, "Exit Blur", 1920, 1080, 1, comp.duration);
exitB.adjustmentLayer = true;
exitB.moveToBeginning();
var exBlur = exitB.property("Effects").addProperty("ADBE Gaussian Blur 2");
exBlur.property("Blurriness").expression =
  'var f=timeToFrames(time); linear(f, DUR-16, DUR, 0, 30);';
exitB.inPoint = secondsAtDurMinus16;

// Exit scale
var exitS = comp.layers.addSolid(bgColor, "Exit Scale", 1920, 1080, 1, comp.duration);
exitS.adjustmentLayer = true;
exitS.moveToBeginning();
var exFx = exitS.property("Effects").addProperty("ADBE Geometry2");
exFx.property("ADBE Geometry2-0003").expression =
  'var f=timeToFrames(time); linear(f, DUR-16, DUR, 100, 106);';
exFx.property("ADBE Geometry2-0004").expression =
  'var f=timeToFrames(time); linear(f, DUR-16, DUR, 100, 106);';
exitS.inPoint = secondsAtDurMinus16;

// Fade to black
var fade = comp.layers.addSolid([0,0,0], "Exit Fade", 1920, 1080, 1, comp.duration);
fade.moveToBeginning();
fade.opacity.setValuesAtTimes(
  [secondsAtDurMinus18, secondsAtDur],
  [0, 100]
);
```

The slight scale-up (100→106%) during exit creates a "zooming into next scene" feel. Combined with blur, it reads as a smooth transition rather than an abrupt cut.
