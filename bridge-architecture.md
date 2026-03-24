# AE Bridge Architecture

How to send ExtendScript commands to After Effects from a Node.js build script.

## File-Polling Pattern

The bridge uses a temp directory where the Node.js script writes `.jsx` command files. A CEP panel running inside AE polls for new commands, executes them, and writes `.json` result files back.

```
Node.js                    Temp Dir                    AE CEP Panel
  │                          │                            │
  ├─ write cmd_001.jsx ─────>│                            │
  │                          │<── poll every 250ms ───────┤
  │                          │                            ├─ execute jsx
  │                          │<── write res_001.json ─────┤
  ├─ read res_001.json <─────│                            │
  ├─ delete res_001.json     │                            │
  │                          │                            │
```

## Bridge Module

```typescript
const BRIDGE_DIR = process.env.AE_TEMP_DIR || 'C:/tmp/ae-mcp-bridge';
const POLL_MS = 250;
const TIMEOUT_MS = 45000;

export async function send(label: string, code: string, critical = false): Promise<any> {
  if (!existsSync(BRIDGE_DIR)) mkdirSync(BRIDGE_DIR, { recursive: true });
  const id = `${Date.now()}_${++cmdCounter}`;
  const cmdFile = join(BRIDGE_DIR, `cmd_${id}.jsx`);
  const resFile = join(BRIDGE_DIR, `res_${id}.json`);

  // Wrap code with helpers and error handling
  const wrapped = `${HELPERS}\ntry {\n${code}\n} catch(e) { "ERR: " + e.toString(); }`;
  writeFileSync(cmdFile, wrapped, 'utf-8');

  // Poll for result
  const start = Date.now();
  while (true) {
    if (existsSync(resFile)) {
      const raw = readFileSync(resFile, 'utf-8');
      unlinkSync(resFile);
      const result = JSON.parse(raw);
      // Check for error prefix
      if (typeof result.data === 'string' && result.data.startsWith('ERR: ')) {
        if (critical) process.exit(1);
        return { success: false, error: result.data };
      }
      return result;
    }
    if (Date.now() - start > TIMEOUT_MS) {
      unlinkSync(cmdFile);
      if (critical) process.exit(1);
      return { success: false, error: 'Timeout' };
    }
    await new Promise(r => setTimeout(r, POLL_MS));
  }
}
```

## Helper Functions

Injected at the top of every command:

```js
// Tick conversion for time-based properties
var TICKS_PER_SECOND = 254016000000;
function __secondsToTicks(s) {
  return Math.round(parseFloat(s) * TICKS_PER_SECOND).toString();
}

// Find a composition by name
function __findComp(name) {
  for (var i = 1; i <= app.project.numItems; i++) {
    try {
      if (app.project.item(i) instanceof CompItem
          && app.project.item(i).name === name) {
        return app.project.item(i);
      }
    } catch(e) {}
  }
  return null;
}

// Find a folder by name
function __findFolder(name) {
  for (var i = 1; i <= app.project.numItems; i++) {
    if (app.project.item(i) instanceof FolderItem
        && app.project.item(i).name === name) {
      return app.project.item(i);
    }
  }
  return null;
}
```

The `try/catch` in `__findComp` is defensive — some project items can throw when accessed.

## Build Script Structure

Each shot is a sequential series of `send()` calls with `sleep()` gaps:

```typescript
async function main() {
  // Step 1: Create comp (critical — abort if this fails)
  await send('Create comp', `...`, true);
  await sleep(300);

  // Step 2: Backgrounds
  await send('Backgrounds', `...`);
  await sleep(200);

  // Step 3-N: Build layers
  // ...

  // Final: Save
  await send('Save', `
    var comp = __findComp("ShotN_Name");
    comp.openInViewer();
    app.project.save();
    "Saved. Layers: " + comp.numLayers;
  `);
}
```

## Timing Guidelines

- **200ms sleep** between normal steps — enough for AE to settle
- **300ms sleep** after comp creation — the project panel needs time to register the new comp
- **1000ms sleep** after heavy shape operations (96+ groups) — AE becomes temporarily unresponsive during digest
- **45s timeout** per command — if AE is genuinely busy (rendering, background task), this is the limit

## Diagnostic Scripts

For debugging composition issues:

```typescript
// Layer analysis — dump all layers with properties
await send('Analyze', `
  var comp = __findComp("ShotN_Name");
  var info = "";
  for (var i = 1; i <= comp.numLayers; i++) {
    var L = comp.layer(i);
    info += i + ": " + L.name;
    info += " | op=" + Math.round(L.opacity.value);
    info += " | pos=[" + Math.round(L.position.value[0]) + "," + Math.round(L.position.value[1]) + "]";
    // ... effects, expressions, blend mode, parent
    info += "\\n";
  }
  info;
`);

// Frame export — visual verification
await send('Export frame', `
  var comp = __findComp("ShotN_Name");
  comp.time = 2.0;
  var file = new File("C:/tmp/shotN_frame.png");
  comp.saveFrameToPng(comp.time, file);
  "Exported to " + file.fsName;
`);
```

Frame export with `saveFrameToPng` is the primary feedback loop for automated composition building. Export at key moments (mid-shot, transitions) to verify visual output.
