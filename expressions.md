# Expression Patterns

Reusable expression structures for AE compositions built via ExtendScript.

## The Universal Fade Pattern

Every animated element follows this structure: fade in, hold/animate, fade out.

```js
layer.opacity.expression = [
  'var f=timeToFrames(time);',
  'var fadeIn=linear(f, START, START+DURATION, 0, PEAK);',
  'var fadeOut=linear(f, DUR-18, DUR, 1, 0);',
  'fadeIn*fadeOut;'
].join('\\n');
```

`fadeOut` multiplies against 1→0, so it scales whatever the current value is. This pattern composes with any other animation.

### With easing
Replace `linear` with `ease` for smoother transitions:
```js
'var fadeIn=ease(f, 4, 22, 0, 100);'
```

### With clamping
When you need hard limits:
```js
'Math.min(fadeIn, fadeOut);'  // caps at the lower value
```

## Pulse / Breathe

Slow sine wave oscillation around a base value.

```js
'var pulse=0.8+0.2*Math.sin(f*0.025);'  // oscillates 0.6–1.0
```

Frequency guide:
| Frequency | Period | Feel |
|-----------|--------|------|
| 0.01 | 628 frames (~10s) | Glacial, ambient |
| 0.025 | 251 frames (~4s) | Gentle breathe |
| 0.04 | 157 frames (~2.6s) | Noticeable pulse |
| 0.12 | 52 frames (~0.9s) | Active heartbeat |

Combine with fade: `fadeIn * pulse * fadeOut`

## Card Entrance Slide

Standard 22-frame ease-in from the right. All card elements use the same timing for cohesion.

```js
layer.position.expression = [
  'var f=timeToFrames(time);',
  'var slideX=ease(f, 0, 22, 500, 0);',
  'value+[slideX, 0]'
].join('\\n');
```

`ease(f, 0, 22, 500, 0)` means: from frame 0 to 22, slide from +500px to +0px with ease in/out.

## Animated Dash Offset

Creates flow/motion along a dashed stroke path.

```js
dashOffset.expression = 'timeToFrames(time) * -2.5';
```

Negative multiplier = dashes flow forward along the path. Adjust magnitude for speed.

## Typing Animation

Reveal characters one-by-one with a blinking cursor.

```js
layer.sourceText.expression = [
  'var fullText="src/services/vectorStore.ts";',
  'var f=timeToFrames(time);',
  'var n=Math.min(Math.floor(linear(f,70,130,0,fullText.length)),fullText.length);',
  'var typed=fullText.substring(0,n);',
  'var cursorOn=(f>=70&&f<130)&&(Math.floor(f/15)%2===0);',
  'typed+(cursorOn?"|":"");'
].join('\\n');
```

Key elements:
- `linear(f, startFrame, endFrame, 0, fullText.length)` maps time to character count
- `Math.floor` snaps to whole characters
- Cursor blinks every 15 frames (4 blinks/sec at 60fps)
- Cursor only shows during typing window

## Conditional Value (Status Change)

Switch expression output at a specific frame.

```js
layer.sourceText.expression = [
  'var f=timeToFrames(time);',
  'if(f>=140) "complete";',
  'else "working";'
].join('\\n');
```

Works for colors too:
```js
fill.property("Color").expression = [
  'var f=timeToFrames(time);',
  'if(f>=140) [0, 0.47, 1];',    // blue
  'else [0.16, 0.82, 0.56];'     // emerald
].join('\\n');
```

## Looping Scale (Ring Pulse)

Cyclic expansion with staggered offset per instance.

```js
var offset = ringIndex * 60;
ring.scale.expression = [
  'var f=timeToFrames(time);',
  'var ringFrame=(f+' + offset + ')%180;',  // 180-frame cycle
  'var s=linear(ringFrame,0,180,30,250);',
  '[s,s]'
].join('\\n');
```

## Wrapping Position (Particle Recycling)

Particles that drift in one direction and seamlessly reappear.

```js
grp.property("Transform").property("Position").expression = [
  'var f=timeToFrames(time);',
  'var x=startX+Math.sin(f*0.015+phase)*30-960;',
  'var y=startY-f*speed-540;',                     // drift upward
  'var wrapped=((y+540)%600)-540+startY-540;',     // wrap at 600px range
  '[x,wrapped]'
].join('\\n');
```

The modulo operation resets the Y position before the particle exits the visible area.

## Wiggle (Noise)

Built-in AE function for randomized animation:
```js
property.expression = 'wiggle(60, 10)';
// 60 = oscillations per second
// 10 = amplitude in property units
```

Useful for film grain noise amount, subtle position jitter, or organic camera shake.

## Building Expressions in TypeScript

When constructing expressions in template literals, remember:
- Escape `\\n` for newlines within the ExtendScript string
- Use `.join('\\n')` on arrays for readable multi-line expressions
- Template literal `${variable}` embeds TypeScript values into the ExtendScript code
- AE expression `linear()` and `ease()` are globally available
- `timeToFrames(time)` converts current time to frame number
