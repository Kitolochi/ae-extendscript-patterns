# UI Mockup Animations

Animated product UI inside compositions — typing cursors, expanding cards, live interface demonstrations.

## Typing Animation with Cursor

A text field that types out character by character with a blinking cursor. Used in the Luma Uni-1 video to show a prompt being entered into their UI.

```js
var textLayer = comp.layers.addText("");
var td = textLayer.sourceText.value;
td.resetCharStyle();
td.fontSize = 24;
td.fillColor = [0.85, 0.85, 0.85];
td.font = "Courier New";  // monospace sells the "typing" feel
textLayer.sourceText.setValue(td);
textLayer.position.setValue([480, 560]);

var fullText = "Create infographic about quarterly revenue";

textLayer.sourceText.expression = [
  'var fullText="' + fullText + '";',
  'var f=timeToFrames(time);',
  'var startFrame=40; var charsPerSec=12;',
  'var elapsed=Math.max(f-startFrame, 0);',
  'var n=Math.min(Math.floor(elapsed*charsPerSec/comp.frameRate), fullText.length);',
  'var typed=fullText.substring(0,n);',
  // Cursor blinks every 15 frames, only during and briefly after typing
  'var typing=n<fullText.length;',
  'var postType=f-startFrame-(fullText.length*comp.frameRate/charsPerSec);',
  'var showCursor=typing||(postType<90&&Math.floor(f/15)%2===0);',
  'typed+(showCursor?"|":"");'
].join('\\n');
```

**12 chars/second** feels natural for a human typing a prompt. Faster than 20 chars/sec reads as autocomplete, not human input. The cursor continues blinking for 90 frames (1.5s at 60fps) after typing finishes, then disappears.

## Product UI Card (Light Theme)

A card with rounded corners on a neutral background, showing a product interface element.

```js
// Card background — rounded rect via shape layer
var card = comp.layers.addShape();
card.name = "UI Card";
var contents = card.property("Contents");
var rectGrp = contents.addProperty("ADBE Vector Group");
var rect = rectGrp.property("Contents").addProperty("ADBE Vector Shape - Rect");
rect.property("Size").setValue([680, 420]);
rect.property("Roundness").setValue(16);  // modern UI radius

var fill = rectGrp.property("Contents").addProperty("ADBE Vector Graphic - Fill");
fill.property("Color").setValue([0.98, 0.98, 0.98]);  // near-white

// Subtle border
var stroke = rectGrp.property("Contents").addProperty("ADBE Vector Graphic - Stroke");
stroke.property("Color").setValue([0.88, 0.88, 0.88]);
stroke.property("Stroke Width").setValue(1);

card.position.setValue([960, 480]);

// Drop shadow for elevation
var shadow = card.property("Effects").addProperty("ADBE Drop Shadow");
shadow.property("Opacity").setValue(0.12);
shadow.property("Direction").setValue(180);
shadow.property("Distance").setValue(4);
shadow.property("Softness").setValue(20);
shadow.property("Shadow Color").setValue([0, 0, 0]);
```

For light-theme UI cards:
- Card fill: [0.96-0.99, 0.96-0.99, 0.96-0.99] — off-white
- Border: 1px, [0.85-0.90] gray
- Shadow: 12% opacity, 4px distance, 20px softness
- Corner radius: 12-16px for modern, 8px for conservative

## Thumbnail Grid Inside Card

Small image placeholders arranged in a grid within the card.

```js
var thumbSize = 120;
var gap = 16;
var startX = 660;  // left edge of card content area
var startY = 380;  // top of thumbnail row

for (var i = 0; i < 3; i++) {
  var thumb = comp.layers.addShape();
  thumb.name = "Thumb " + (i + 1);
  var tc = thumb.property("Contents");
  var tg = tc.addProperty("ADBE Vector Group");
  var tr = tg.property("Contents").addProperty("ADBE Vector Shape - Rect");
  tr.property("Size").setValue([thumbSize, thumbSize]);
  tr.property("Roundness").setValue(8);

  var tf = tg.property("Contents").addProperty("ADBE Vector Graphic - Fill");
  // Alternate subtle colors to differentiate placeholders
  var colors = [[0.85, 0.88, 0.92], [0.88, 0.85, 0.90], [0.85, 0.90, 0.86]];
  tf.property("Color").setValue(colors[i]);

  thumb.position.setValue([startX + i * (thumbSize + gap), startY]);
}
```

If actual screenshots or renders exist, import them and use as layer source instead of colored placeholders. The shape layer approach works for mockup-style demonstrations where the content matters less than the UI chrome.

## Card Entrance Animation

Slide up from below with opacity fade + slight scale overshoot.

```js
card.position.expression = [
  'var f=timeToFrames(time);',
  'var slideY=ease(f, 0, 24, 80, 0);',  // 80px upward slide
  'value+[0, slideY]'
].join('\\n');

card.opacity.expression = [
  'var f=timeToFrames(time);',
  'ease(f, 0, 18, 0, 100);'
].join('\\n');

card.scale.expression = [
  'var f=timeToFrames(time);',
  'var s=ease(f, 0, 24, 96, 101);',      // 96% → 101%
  'if(f>24) s=ease(f, 24, 32, 101, 100);', // settle to 100%
  '[s,s]'
].join('\\n');
```

The slide (80px), fade (18 frames), and scale overshoot (101% → 100%) should all complete within 32 frames (0.5s). Stagger child elements by 4-6 frames each for a cascade effect.

## Text Input Field

A text field with placeholder text that transforms into typed text.

```js
// Field background
var field = comp.layers.addShape();
field.name = "Input Field";
var fc = field.property("Contents");
var fg = fc.addProperty("ADBE Vector Group");
var fr = fg.property("Contents").addProperty("ADBE Vector Shape - Rect");
fr.property("Size").setValue([500, 48]);
fr.property("Roundness").setValue(8);
var ff = fg.property("Contents").addProperty("ADBE Vector Graphic - Fill");
ff.property("Color").setValue([0.95, 0.95, 0.95]);
var fs = fg.property("Contents").addProperty("ADBE Vector Graphic - Stroke");
fs.property("Color").setValue([0.80, 0.80, 0.80]);
fs.property("Stroke Width").setValue(1);
field.position.setValue([960, 600]);

// Focus state — border color change
fs.property("Color").expression = [
  'var f=timeToFrames(time);',
  'var focusFrame=35;',
  'if(f>=focusFrame) [0.2, 0.5, 1];',   // blue focus ring
  'else [0.80, 0.80, 0.80];'             // default gray
].join('\\n');
fs.property("Stroke Width").expression = [
  'var f=timeToFrames(time);',
  'if(f>=35) 2; else 1;'
].join('\\n');
```

The focus state change (gray → blue border, 1px → 2px) at the moment typing starts sells the interaction. Without it, the typing feels disconnected from the UI.

## Button with Hover State

```js
var btn = comp.layers.addShape();
btn.name = "Submit Button";
var bc = btn.property("Contents");
var bg = bc.addProperty("ADBE Vector Group");
var br = bg.property("Contents").addProperty("ADBE Vector Shape - Rect");
br.property("Size").setValue([140, 44]);
br.property("Roundness").setValue(8);
var bf = bg.property("Contents").addProperty("ADBE Vector Graphic - Fill");
bf.property("Color").expression = [
  'var f=timeToFrames(time);',
  'var hover=150; var click=158;',
  'if(f>=click) [0.15, 0.4, 0.85];',     // pressed: darker
  'else if(f>=hover) [0.25, 0.55, 1];',  // hover: slightly brighter
  'else [0.2, 0.5, 1];'                   // default blue
].join('\\n');
btn.position.setValue([960, 680]);

// Scale on click
btn.scale.expression = [
  'var f=timeToFrames(time);',
  'var click=158;',
  'if(f>=click&&f<162) {var s=ease(f,158,160,100,96); [s,s];}',
  'else if(f>=162&&f<166) {var s=ease(f,162,166,96,100); [s,s];}',
  'else [100,100];'
].join('\\n');
```

The 4-frame scale-down to 96% on click, then 4-frame bounce back, mimics CSS `transform: scale(0.96)` on `:active`. Keeps the interaction feeling responsive even in a pre-rendered composition.
