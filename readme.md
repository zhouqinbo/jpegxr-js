# jpegxr-js

JPEG XR decoding in JavaScript, with fast sub-resolution decode.

Based on [bvibber/jpegxr](https://github.com/bvibber/jpegxr) by Brooke Vibber (commit `3e8b298`), a wrapper around
the jxrlib codec open-sourced by Microsoft. This version is not endorsed by the original authors. Decoding only.
Already in use in a real project.

# Usage

**[Download jpegxr.js](https://github.com/zhouqinbo/jpegxr-js/raw/main/dist/jpegxr.js)** and copy it into your
project. No build, no npm: a single ES module file (185 KB) with the WebAssembly binary inlined.

```js
import jpegxr from './jpegxr.js';

const bytes = new Uint8Array(await (await fetch('image.jxr')).arrayBuffer());
const codec = await jpegxr();
const image = codec.decode(bytes);              // full size
const quarter = codec.decodeAtScale(bytes, 4);  // 1/4 size, much faster
```

`decode` returns:

```js
{
    width: number,
    height: number,
    pixelInfo: {
        channels: number, // 3, 4 etc
        colorFormat: string, // "RGB" etc
        bitDepth: string, // "8", "32Float" etc
        bitsPerPixel: number, // 24, 32, 128 etc
        hasAlpha: boolean,
        premultipliedAlpha: boolean,
        bgr: boolean, // indicates RGB has blue channel first, not red
    },
    bytes: Uint8Array
}
```

Exceptions are thrown for invalid data. `bytes` has its own buffer and does not need to be freed.

# Sub-resolution decode

`decodeAtScale(bytes, scale)` returns the same structure at `ceil(width / scale)` by `ceil(height / scale)` pixels;
`scale` is 1, 2, 4, 8 or 16. jxrlib's inverse transform skips the high-frequency subbands, so this is much faster than
a full decode followed by a downscale. Useful for thumbnails and fit-to-window views of large images.

Measured on `samples/panel-hdr.jxr` (3440 x 1440), average colour relative to the full decode:

| scale | size | time | average R / G / B |
|---|---|---|---|
| 1 | 3440 x 1440 | 334 ms | 1.000 / 1.000 / 1.000 |
| 2 | 1720 x 720 | 262 ms | 1.036 / 1.038 / 1.043 |
| 4 | 860 x 360 | 26 ms | 0.973 / 0.984 / 0.990 |
| 8 | 430 x 180 | 23 ms | 0.970 / 0.982 / 0.989 |
| 16 | 215 x 90 | 5 ms | 0.921 / 0.951 / 0.969 |

# Building

`dist/jpegxr.js` is built with emscripten 5.0.7 (emsdk needs Python 3.10 or newer):

```sh
git clone https://github.com/emscripten-core/emsdk.git ~/emsdk
~/emsdk/emsdk install 5.0.7 && ~/emsdk/emsdk activate 5.0.7
source ~/emsdk/emsdk_env.sh
make
cp wasm/jpegxr.js dist/jpegxr.js && printf '\nexport default jpegxr;\n' >> dist/jpegxr.js
shasum -a 256 dist/jpegxr.js   # c83efc7dc2b0077cffab7896bbcba40363f0b261c44e8d1f03b5074115c7818b
```

Tests, after `make`: `npm test` decodes the sample at full size; `node wasm/test-subres.js` checks every scale.

# Authors and license

- jxrlib: Copyright (c) 2009, Microsoft
- JS wrapper and tweaks to the C code: Copyright (c) Brooke Vibber `<bvibber@pobox.com>`
- Sub-resolution decode and the newer-emscripten fix: Copyright (c) 2026 qinbo zhou

BSD license; see [`license.md`](license.md) and [`NOTICE.md`](NOTICE.md). Keep these notices when you redistribute.
