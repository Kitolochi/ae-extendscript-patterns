# AE ExtendScript Patterns

Patterns and gotchas for building After Effects compositions programmatically via ExtendScript, learned through automated video production with a CEP bridge.

These docs emerged from building 10+ shots for a product trailer entirely through code — no manual AE interaction. Each pattern was discovered through trial, error, and exported frame verification.

## Guides

### Core Patterns (from production builds)

| Document | What it covers |
|----------|---------------|
| [gotchas.md](gotchas.md) | API traps, property name mismatches, null references, range errors |
| [backgrounds.md](backgrounds.md) | Radial gradients, color washes, scan lines, aurora glow |
| [atmospheric-elements.md](atmospheric-elements.md) | Glow orbs, concentric rings, floating particles, code particles, data flow lines |
| [card-construction.md](card-construction.md) | Glassmorphism cards, rounded corners, border strokes, shadows, shimmer, progress bars |
| [3d-perspective.md](3d-perspective.md) | Precomp approach, camera setup, ground reflections, edge lighting, why not Material Options |
| [expressions.md](expressions.md) | Fade patterns, pulse/breathe, typing animation, conditional values, looping, wrapping |
| [post-effects.md](post-effects.md) | Vignette, film grain, entry/exit transitions |
| [bridge-architecture.md](bridge-architecture.md) | File-polling bridge, helper functions, build script structure, diagnostics |

### Advanced Techniques (from studying professional launch videos)

| Document | What it covers |
|----------|---------------|
| [particle-systems.md](particle-systems.md) | Sparse fields, 4-phase constellation (band→burst→converge→sphere), dual-color hemispheres, dispersal, precise timing values |
| [title-treatments.md](title-treatments.md) | Embossed/carved text, reveal wipes, scale-from-zero, tracking expand, light sweep |
| [ui-mockups.md](ui-mockups.md) | Typing animation, product UI cards, thumbnail grids, input fields, button states |
| [hud-overlays.md](hud-overlays.md) | Leader lines, numerical readouts, corner brackets, scan lines, ambient grids |
| [composition-structure.md](composition-structure.md) | Bookend motifs, split composition, content mosaic, subtitle bars, scene transitions |
| [generation-visualization.md](generation-visualization.md) | Progressive denoising, scan-line reveals, glitch artifacts, "Generating..." status |

## Key Principles

**Shape layers over solids.** Solids show rectangular edges through blur. Shape layer ellipses don't. Use solids only for full-frame layers (BG, adjustment layers, vignettes).

**Precomp before 3D.** Individual 3D layers rotate around their own anchor points. Group layers into a precomp first, then make the single precomp 3D.

**Overlay effects over AE lights.** Material Options + point lights darken content unpredictably. ADD-blend shape layers give you precise control over ambient glow.

**4-point bezier for ellipses.** The kappa constant 0.5522847498 produces a mathematically correct circle from 4 points. Multi-point trig loops fail in ExtendScript.

**Every element fades out.** Multiply by `linear(f, DUR-18, DUR, 1, 0)` so nothing is left hanging at the end of a shot.

## Stack

- **AE 2025** with CEP panel (MCPBridgeCEP)
- **Node.js + tsx** for build scripts
- **File-polling bridge** at `C:/tmp/ae-mcp-bridge/`
- **ExtendScript ES3** — `var` only, no `let`/`const`, no arrow functions

## References

Visual techniques studied from professional product launch videos:
- **Luma Labs Uni-1** (Mar 2026) — particle constellation formation, embossed title cards, typing animation UI, HUD data overlays, content mosaic grid, bookend narrative structure

## Context

Built for a product video trailer using a pipeline that goes: Remotion (React) → rendered clips → After Effects (via this bridge) → Premiere Pro (via MCP). The AE stage adds motion graphics, 3D perspective, and post-processing that Remotion can't match.
