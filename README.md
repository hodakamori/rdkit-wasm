# rdkit-wasm

RDKit's ETKDG conformer embedding and MMFF94/UFF minimisation compiled to
WebAssembly with Emscripten. It was written for
[megane](https://github.com/megane-labs/megane) Builder's "embed a flat sketch
in 3D" feature and lived in that repository's `rdkit-wasm/` directory until it
was split out here. It exists because the official RDKit.js (`@rdkit/rdkit`,
MinimalLib) exposes only 2D coordinate generation (`set_new_coords`), and a
pure-JS alternative (openchemlib-js) measured 10–50× slower than RDKit on the
same molecules.

The wrapper is intentionally tiny (`src/rdkit_embed.cpp`, three functions)
and RDKit itself is built unmodified from a pinned release tag, so upgrading
RDKit is a one-line change to `RDKIT_TAG` in `scripts/build.sh`.

## Build

Requirements: git, curl, cmake ≥ 3.20, ninja (optional), a host C++ compiler
for Boost's `bootstrap.sh`, Python 3 (emsdk), Node 22+. Eigen headers are
taken from `/usr/include/eigen3` when present (`apt install libeigen3-dev`)
and cloned otherwise.

```bash
bash scripts/build.sh          # deps → rdkit → wrapper → test
bash scripts/build.sh rdkit    # just rebuild the RDKit libraries
bash scripts/build.sh wrapper  # just relink the wrapper into dist/
node test/smoke.mjs
```

The same steps are exposed as npm scripts (`npm run build`, `npm run
build:rdkit`, `npm run build:wrapper`, `npm test`).

Everything the build downloads or compiles lives in `.deps/` (override with
`RDKIT_WASM_DEPS_DIR`), which the CI workflow caches so only the wrapper
relinks on a normal run. Only the four RDKit targets the wrapper links
(`DistGeomHelpers`, `ForceFieldHelpers`, `FileParsers`, `SmilesParse`) and
their dependencies are compiled; drawing, fingerprints, reactions, InChI,
coordgen and the other externals are switched off, and RDKit's configure-time
downloads (RingDecomposerLib, …) are disabled so the build also works behind
egress policies that block GitHub release archives.

Outputs: `dist/rdkit-embed.mjs` (Emscripten ES-module glue) and
`dist/rdkit-embed.wasm`. Neither is committed; the CI workflow
(`.github/workflows/build.yml`) builds them on every pull request and push to
`main` and uploads them as the `rdkit-embed-wasm-<RDKIT_TAG>` artifact.

## API

```ts
import { loadRDKitEmbed } from "@megane-labs/rdkit-embed-wasm";

const rdkit = await loadRDKitEmbed({ locateFile: (f) => new URL(`./dist/${f}`, import.meta.url).href });
rdkit.version(); // "2026.03.6"

const { molblocks, energies, converged, forceField, warnings } = rdkit.embed(molfileFromKetcher, {
  numConfs: 1,          // ETKDG conformers to generate
  forceField: "MMFF94s", // "MMFF94s" | "MMFF94" | "UFF" | "none"; MMFF falls back to UFF when untyped
  maxIters: 500,
  randomSeed: 42,       // the loader's default; -1 lets RDKit pick
  embedParams: { useRandomCoords: true }, // any RDKit EmbedParameters override
});
```

`embed` accepts a SMILES or an MDL mol block (auto-detected). Hydrogens are
added before embedding (`addHs: true`) and kept in the output unless
`removeHs: true`. Stereo from wedge bonds in the mol block is honoured by ETKDG
(`enforceChirality`). The result is one V2000 mol block per conformer, which
any MOL parser (megane's included) reads directly.

The call is synchronous and takes tens to hundreds of milliseconds for
drug-sized molecules, so run it in a Web Worker in the browser.

## Layout

| Path | What it is |
| --- | --- |
| `src/rdkit_embed.cpp` | The embind wrapper: parse → add Hs → ETKDG → MMFF/UFF → mol blocks |
| `CMakeLists.txt` | Links the wrapper against the RDKit static libraries; Emscripten flags live here |
| `scripts/build.sh` | Fetches the toolchain and dependencies, builds RDKit, links the wrapper, runs the smoke test |
| `index.mjs` / `index.d.ts` | The loader and its TypeScript types |
| `test/smoke.mjs` | Loads `dist/`, embeds a few molecules and asserts the geometry is 3D |
| `test/bench.mjs` | Timing harness (nothing asserted) for comparing against RDKit in Python |

This repository is deliberately separate from megane: the build takes tens of
minutes and needs emsdk, Boost and a C++ compiler that the rest of megane never
touches; the artifact is a versioned binary that megane consumes as a
dependency rather than rebuilds; and RDKit's release cadence is independent of
megane's.

## Licensing

The wrapper is MIT (see `LICENSE`). RDKit is BSD-3-Clause, Boost is BSL-1.0,
zlib is the zlib licence and Eigen is MPL-2.0 (header only); all are
compatible with the MIT licence of this package and of megane. Open Babel was
ruled out for being GPL-2.
