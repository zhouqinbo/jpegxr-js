# dist/jpegxr.js

The prebuilt decoder: a single ES module file with the WebAssembly binary inlined. Usage and build steps are in
[`../readme.md`](../readme.md).

| | |
|---|---|
| Built with | emscripten 5.0.7 and the repository's `Makefile`, then `export default jpegxr;` appended |
| Size | 185,219 bytes |
| SHA-256 | `c83efc7dc2b0077cffab7896bbcba40363f0b261c44e8d1f03b5074115c7818b` |

Contains code of jxrlib (Copyright (c) 2009, Microsoft) and of jpegxr (Copyright (c) Brooke Vibber) under the BSD
license in [`../license.md`](../license.md); see [`../NOTICE.md`](../NOTICE.md). Keep those notices with any copy you
redistribute.
