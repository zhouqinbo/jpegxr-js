# dist/jpegxr.js: prebuilt, ready to use

A single-file build with the WebAssembly binary inlined (no separate `.wasm`), already an ES module
(`export default jpegxr;` is the last line). Copy it into your project and import it; no npm and no emscripten needed.

```js
import jpegxr from './jpegxr.js';
const codec = await jpegxr();
const quarter = codec.decodeAtScale(bytes, 4);
```

| | |
|---|---|
| Source | this repository's code (bvibber/jpegxr 3e8b298 + the sub-resolution change 2a2e4d0) |
| Built | 2026-05-06 with emscripten 5.0.7, `make` with the repository's Makefile unchanged (`-O2`, `MODULARIZE=1`, `EXPORT_NAME=jpegxr`, `SINGLE_FILE=1`), then `export default jpegxr;` appended |
| Size | 185,219 bytes |
| SHA-256 | `c83efc7dc2b0077cffab7896bbcba40363f0b261c44e8d1f03b5074115c7818b` |
| Reproducible | rebuilding from this repository with emscripten 5.0.7 gives a byte-identical file (checked 2026-11-22; steps in `../readme.md`, "Building (JS)") |
| In use | the same file is in use in a real project |

The file contains code of jxrlib (Copyright (c) 2009, Microsoft) and of the jpegxr wrappers (Copyright (c) Brooke
Vibber), under the BSD license in `../license.md`; see `../NOTICE.md`. Keep those notices with any copy you
redistribute.
