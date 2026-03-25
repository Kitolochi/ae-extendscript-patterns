# Title Treatments

Techniques for making title text feel physical and cinematic rather than flat.

## Embossed / Carved Text

Text that appears stamped into a textured surface. The Luma Uni-1 video opens with "UNI-1" embossed into what looks like stone or leather.

### Approach: Bevel + Fractal Texture

```js
// 1. Create the textured background surface
var surface = comp.layers.addSolid([0.12, 0.11, 0.10], "Surface", 1920, 1080, 1, comp.duration);
var fractal = surface.property("Effects").addProperty("ADBE Fractal Noise");
fractal.property("Fractal Type").setValue(3);        // Turbulent Basic
fractal.property("Contrast").setValue(120);
fractal.property("Brightness").setValue(-20);
fractal.property("Transform").property("Scale").setValue(80);
fractal.property("Complexity").setValue(6);

// 2. Create the title text
var title = comp.layers.addText("UNI-1");
var td = title.sourceText.value;
td.resetCharStyle();
td.fontSize = 200;
td.fillColor = [0.15, 0.14, 0.13];  // near-match to surface tone
td.font = "Arial";                    // heavy weight works best
td.justification = ParagraphJustification.CENTER_JUSTIFY;
title.sourceText.setValue(td);
title.position.setValue([960, 540]);

// 3. Apply Bevel and Emboss for the carved look
var bevel = title.property("Effects").addProperty("ADBE Bevel Alpha");
// Alternative: use "ADBE Bevel And Emboss" if available
bevel.property("Edge Thickness").setValue(3);
bevel.property("Light Angle").setValue(135);
bevel.property("Light Color").setValue([1, 0.95, 0.9]);   // warm highlight
bevel.property("Light Intensity").setValue(0.4);

// 4. Drop shadow for inset depth
var shadow = title.property("Effects").addProperty("ADBE Drop Shadow");
shadow.property("Opacity").setValue(0.6);
shadow.property("Direction").setValue(135);
shadow.property("Distance").setValue(3);
shadow.property("Softness").setValue(6);
shadow.property("Shadow Color").setValue([0, 0, 0]);
```

The fill color of the text should near-match the surface color. The bevel highlight and shadow create the illusion of depth. A brighter fill looks stamped outward (raised); a darker fill looks carved inward (embossed).

### Alternative: Displacement Map

For a more organic carved effect, use the text layer as a displacement map on the surface:

```js
// Use text as a track matte + displacement
var textPrecomp = /* precomp containing white text on black */;
var displace = surface.property("Effects").addProperty("ADBE Displacement Map");
displace.property("Displacement Map Layer").setValue(textPrecompIndex);
displace.property("Max Horizontal Displacement").setValue(8);
displace.property("Max Vertical Displacement").setValue(8);
displace.property("Displacement Map Behavior").setValue(2);  // Tile
```

Displacement gives a more natural warp that interacts with the fractal texture. The text edges bend the surface rather than sitting on top of it.

## Reveal Wipe

Title text revealed by a luminance wipe — dark to light sweep across the frame.

```js
var title = comp.layers.addText("PRODUCT NAME");
// ... font setup ...

var wipe = title.property("Effects").addProperty("ADBE Linear Wipe");
wipe.property("Transition Completion").expression = [
  'var f=timeToFrames(time);',
  'linear(f, 20, 50, 100, 0);'
].join('\\n');
wipe.property("Wipe Angle").setValue(270);   // left to right
wipe.property("Feather").setValue(80);       // soft edge
```

High feather (80+) creates a gradient reveal rather than a hard edge. The text materializes from one side, suggesting emergence.

## Scale-From-Zero Title

Title starts at 0% scale, punches up to 102%, then settles to 100%. Combined with blur clear.

```js
title.scale.expression = [
  'var f=timeToFrames(time);',
  'var s=ease(f,0,18,0,102);',           // 0→102% over 18 frames
  'if(f>18) s=ease(f,18,26,102,100);',   // 102→100% settle
  '[s,s]'
].join('\\n');

var blur = title.property("Effects").addProperty("ADBE Gaussian Blur 2");
blur.property("Blurriness").expression = [
  'var f=timeToFrames(time);',
  'ease(f,0,14,30,0);'
].join('\\n');
```

The 2% overshoot and settle gives the appearance of physical weight. Without it, the scale-up feels digital and weightless.

## Tracking Expand

Letters start tight, then track outward. Mimics high-end typography animation.

```js
title.sourceText.expression = [
  'var f=timeToFrames(time);',
  'var base=value;',
  'var tracking=ease(f, 10, 40, 0, 800);',
  'base.setTracking(tracking);',
  'base;'
].join('\\n');
```

Note: `setTracking` modifies the TextDocument's tracking property within an expression. Values are in thousandths of an em (800 = wide spacing). Combine with opacity fade for a "breathing into existence" feel.

## Light Sweep on Text

A bright diagonal line that sweeps across the title surface. Adds metallic sheen.

```js
// Ramp effect angled across text, animated position
var ramp = title.property("Effects").addProperty("ADBE Ramp");
ramp.property("Start of Ramp").expression = [
  'var f=timeToFrames(time);',
  'var x=linear(f, 30, 60, -400, 2300);',
  '[x, 0]'
].join('\\n');
ramp.property("End of Ramp").expression = [
  'var f=timeToFrames(time);',
  'var x=linear(f, 30, 60, -200, 2500);',
  '[x, 1080]'
].join('\\n');
ramp.property("Start Color").setValue([1, 1, 1]);    // white
ramp.property("End Color").setValue([1, 1, 1]);
ramp.property("Ramp Shape").setValue(2);              // Linear
ramp.property("ADBE Ramp Blend With Original").setValue(0.85);  // range 0-1

title.blendingMode = BlendingMode.ADD;
```

Set blend with original to 0.85 so the sweep only adds a subtle highlight (15% contribution) rather than washing out the text.
