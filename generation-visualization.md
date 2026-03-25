# Generation Visualization

Techniques for showing an AI model's generation process as a visual effect — progressive denoising, scan-line reveals, and noisy-to-clean transitions. Discovered in the Luma Uni-1 video at t=30 where the "Generating..." state shows a partially-rendered image with glitch artifacts.

## Progressive Denoising Effect

Simulates an image forming from noise to clarity, like a diffusion model's denoising process.

```js
// Start with a noisy version of the final image, progressively reveal the clean version
// Layer stack (top to bottom):
// 1. Clean final image (opacity ramps from 0→100)
// 2. Noisy overlay (opacity ramps from 100→0)
// 3. Black background

// Clean image layer
var clean = comp.layers.add(finalImageSource);
clean.name = "Clean Output";
clean.opacity.expression = [
  'var f=timeToFrames(time);',
  'ease(f, 30, 120, 0, 100);'  // 1.5s denoising transition
].join('\\n');

// Noise overlay
var noisy = comp.layers.addSolid([0.5, 0.5, 0.5], "Noise Overlay", 1920, 1080, 1, comp.duration);
noisy.blendingMode = BlendingMode.OVERLAY;
var noise = noisy.property("Effects").addProperty("ADBE Fractal Noise");
noise.property("Fractal Type").setValue(5);       // Dynamic Progressive
noise.property("Contrast").setValue(200);
noise.property("Brightness").setValue(-30);
noise.property("Transform").property("Scale").setValue(150);
noise.property("Evolution").expression = 'time*500';  // animated noise

// Noise fades out as image forms
noisy.opacity.expression = [
  'var f=timeToFrames(time);',
  'ease(f, 30, 120, 80, 0);'
].join('\\n');
```

The fractal noise at high contrast with animated evolution creates the impression of an image resolving from random noise. The overlap period (where both noise and clean image are partially visible) is where the effect sells.

## Scan Line Reveal

Horizontal lines sweep through the image, revealing it strip by strip. Mimics a rasterization or row-by-row generation process.

```js
// Adjustment layer with a venetian-blind-style mask reveal
var reveal = comp.layers.addSolid([0, 0, 0], "Scan Reveal", 1920, 1080, 1, comp.duration);
reveal.adjustmentLayer = true;

// Use CC Jaws (or similar line-based transition)
// Alternative: multiple horizontal strip masks with staggered reveals
var lineCount = 12;
var lineHeight = Math.ceil(1080 / lineCount);

for (var i = 0; i < lineCount; i++) {
  var mask = reveal.property("Masks").addProperty("ADBE Mask Atom");
  var ms = new Shape();
  var y = i * lineHeight;
  ms.vertices = [[0, y], [1920, y], [1920, y + lineHeight], [0, y + lineHeight]];
  ms.closed = true;
  mask.property("ADBE Mask Shape").setValue(ms);
  mask.maskMode = MaskMode.ADD;

  // Staggered opacity per strip
  var delay = i * 3;  // 3 frames between each strip
  mask.maskOpacity.expression = [
    'var f=timeToFrames(time);',
    'linear(f, ' + (40 + delay) + ', ' + (52 + delay) + ', 0, 100);'
  ].join('\\n');
}
```

## Glitch/Artifact Overlay

Digital corruption artifacts layered on top of forming content. The Luma video shows horizontal displacement glitches and color channel separation during generation.

```js
// Horizontal displacement strips
var glitch = comp.layers.addShape();
glitch.name = "Glitch Strips";
var gc = glitch.property("Contents");

for (var i = 0; i < 8; i++) {
  var gg = gc.addProperty("ADBE Vector Group");
  var gr = gg.property("Contents").addProperty("ADBE Vector Shape - Rect");
  var stripH = 4 + Math.random() * 20;
  gr.property("Size").setValue([1920, stripH]);
  var gf = gg.property("Contents").addProperty("ADBE Vector Graphic - Fill");
  // Cyan or magenta channel color
  var isRed = Math.random() > 0.5;
  gf.property("Color").setValue(isRed ? [1, 0, 0.3] : [0, 0.8, 1]);
  gf.property("Opacity").setValue(20 + Math.random() * 30);

  var baseY = Math.random() * 1080;
  var offsetX = (Math.random() - 0.5) * 200;  // horizontal displacement

  gg.property("Transform").property("Position").expression = [
    'var f=timeToFrames(time);',
    // Appear for random bursts
    'var phase=' + (Math.random() * 200) + ';',
    'var active=Math.sin(f*0.3+phase)>0.7;',
    'var x=960+' + offsetX + '*(active?1:0);',
    'var y=' + baseY + '+Math.sin(f*0.1+phase)*30;',
    '[x,y]'
  ].join('\\n');

  gg.property("Transform").property("Opacity").expression = [
    'var f=timeToFrames(time);',
    'var phase=' + (Math.random() * 200) + ';',
    'var active=Math.sin(f*0.3+phase)>0.7;',
    // Fade out over time as generation completes
    'var fade=linear(f, 30, 100, 1, 0);',
    '(active?50:0)*fade;'
  ].join('\\n');
}
```

The glitch strips appear in random bursts (`sin > 0.7` creates intermittent visibility) and fade out as the "generation" completes. Horizontal displacement of 100-200px simulates data corruption. Cyan and magenta colors mimic RGB channel separation.

## "Generating..." Status Indicator

Text label with animated ellipsis showing the process is active.

```js
var status = comp.layers.addText("Generating");
var td = status.sourceText.value;
td.resetCharStyle();
td.fontSize = 16;
td.fillColor = [0.7, 0.7, 0.7];  // light gray
td.font = "Helvetica Neue";
td.tracking = 50;
status.sourceText.setValue(td);
status.position.setValue([140, 80]);  // upper-left corner

// Animated dots
status.sourceText.expression = [
  'var f=timeToFrames(time);',
  'var dots=Math.floor(f/20)%4;',  // cycles through 0,1,2,3 dots
  'var d="";',
  'for(var i=0;i<dots;i++) d+=".";',
  '"Generating"+d;'
].join('\\n');
```

The dot cycle every 20 frames (3 cycles/sec at 60fps) is fast enough to feel active without being distracting.

## Combined Generation Sequence

The full pattern chains these elements: noise → scan reveal → glitch artifacts → clean result. In the Luma video, this plays out over ~3 seconds (t=30 to t=33).

```
Timeline:
f0-f30:   Full noise, "Generating..." label visible
f30-f60:  Scan lines begin revealing, noise starts fading
f60-f90:  Image 70% visible, glitch artifacts at peak
f90-f120: Glitch fades, image fully resolved
f120+:    Clean image holds, "Generating..." changes to complete state
```

Each element should be on its own layer for independent timing control. The noise overlay sits above the clean image; the glitch strips sit above both; the status text sits above everything.

## Luma Uni-1 Reference (t=30)

The generation visualization at t=30 shows:
- **"Generating..."** text in upper-left, small white sans-serif
- **Partially rendered image**: Golden Gate Bridge forming from noise/scan artifacts
- **Blue/white light streaks**: Diagonal energy lines across the forming image
- **Progressive clarity**: Center of image resolves first, edges last
- **Duration**: ~3 seconds from first visible content to clean output
- **Transition out**: Clean image holds, then camera pans horizontally across the result panels
