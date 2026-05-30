# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development (esbuild, fast incremental)
npm run dev:viewer          # viewer app only — fastest iteration
npm run dev:apps            # all apps
npm run dev                 # default (no apps/examples selected; prompts)
npm run dev:all             # all apps, examples, browser-tests
npm run serve               # static HTTP server on :1338

# Build
npm run build               # full build: apps + library (tsc + esbuild)
npm run build:apps          # esbuild apps only
npm run build:lib           # tsc → lib/ (ESM) + lib/commonjs/ (CJS)
npm run rebuild             # clean + full build

# Test & lint
npm run test                # lint + jest (also installs gl dependency)
npm run jest                # jest only
npm run lint                # ESLint
npm run lint-fix            # ESLint with auto-fix

# Run a single test file
npx jest src/path/to/spec.ts

# Servers (require built lib/commonjs/)
npm run model-server
npm run volume-server-test
npm run plugin-state
```

Requires Node >= 22.

## Architecture

Mol* is a modular TypeScript library for 3D visualization and analysis of macromolecular structures. Source lives in `src/` as ~27 independent `mol-*` packages with strict layering — lower packages must not import from higher ones.

### Core layers (bottom → top)

| Package | Role |
|---|---|
| `mol-task` | Async task abstraction with progress/cancellation via `RuntimeContext` |
| `mol-data` | Low-level collections: columns, integer sets, sorted arrays |
| `mol-math` | Linear algebra, spatial grids, geometry algorithms |
| `mol-io` | Format parsers: CIF/BinaryCIF, PDB, SDF, volume maps (CCP4/DSN6/etc.) |
| `mol-model` | Core data structures (`Structure`, `Unit`, `Residue`, …) and MolQL query engine |
| `mol-model-formats` | Bridges `mol-io` parsers into `mol-model` structures |
| `mol-model-props` | Custom per-model property definitions (computed or fetched) |
| `mol-geo` | Geometry primitives and mesh generation |
| `mol-theme` | Color/size themes for structures, volumes, and shapes |
| `mol-repr` | Representations: `StructureRepresentation`, `VolumeRepresentation`, `ShapeRepresentation` |
| `mol-gl` | Thin WebGL2 wrapper: `WebGLContext`, render objects, render passes |
| `mol-canvas3d` | `Canvas3D` — camera, picking, passes (SSAO, outline, depth-of-field) |
| `mol-script` | MolQL scripting language + transpilers |
| `mol-state` | Immutable state tree: `StateObject`, `StateTransformer`, `StateAction` |
| `mol-plugin` | `PluginContext` — assembles canvas, state, behaviors, managers |
| `mol-plugin-state` | State transformers/builders/managers for plugin-level state |
| `mol-plugin-ui` | React 18 UI components (uses `mol-plugin` as peer) |
| `mol-util` | Shared utilities (observables, color, debug, assets, …) |

### Other directories

- `apps/` — bundled applications (viewer, mvs-stories)
- `extensions/` — optional plugin extensions (DNATCO, G3D, volumes-and-segmentations, …)
- `servers/` — Express.js servers: model (structure queries), volume (density data), plugin-state
- `cli/` — Node CLI tools: `cif2bcif`, `cifschema`, `mvs-render`, `mvs-validate`
- `examples/` — standalone integration examples
- `tests/` — integration/browser tests

### State system

The plugin uses a reactive state tree (`mol-state`). Data flows through `StateTransformer`s — pure functions that transform one `StateObject` into another. `StateBuilder` provides a fluent API to compose these transforms. Undo/redo and serialization come for free.

### Build outputs

- `lib/` — ESM (via `tsc`)
- `lib/commonjs/` — CommonJS (via `tsc --build tsconfig.commonjs.json` + `tsc-alias`)
- `build/` — bundled apps (via esbuild)

### TypeScript

Strict mode (`strict: true`), target ES2018, `moduleResolution: bundler`. Path aliases like `Mol/mol-*` are resolved by `tsc-alias` post-build.
