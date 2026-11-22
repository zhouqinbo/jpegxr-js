# jpegxr-js

Decode JPEG XR images in the browser. Fast, one file.

## What is JPEG XR

An image format like JPEG, but it can also store:

- HDR, up to 32-bit float per channel
- lossless images
- transparency

Browsers cannot display it.

## Where you meet it

| Source | Example |
|---|---|
| `.jxr` files | HDR screenshots from the Xbox Game Bar on Windows |
| Zeiss CZI files | Microscopy whole-slide images, stored as JPEG XR tiles |

## Why jpegxr-js

- **Works in any browser.** One file. No build, no npm.
- **Fast thumbnails.** Decode at 1/2 to 1/16 size directly, over 10x faster at 1/4 and smaller.
- **In real use.** Already decoding images in a real project.

# Usage

1. **[Download jpegxr.js](https://github.com/zhouqinbo/jpegxr-js/raw/main/dist/jpegxr.js)** (185 KB, an ES module).
2. Copy it into your project.
3. Import it and decode, in a `<script type="module">` or with a bundler:

```js
import jpegxr from './jpegxr.js';

const bytes = new Uint8Array(await (await fetch('image.jxr')).arrayBuffer());
const codec = await jpegxr();
const image = codec.decode(bytes);              // full size
const quarter = codec.decodeAtScale(bytes, 4);  // 1/4 size, much faster
```

In Node.js: delete the last line of the file (`export default jpegxr;`), save it as `jpegxr.cjs`, and load it with
`const jpegxr = require('./jpegxr.cjs');`.

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
        premultipledAlpha: boolean, // spelled this way
        bgr: boolean, // indicates RGB has blue channel first, not red
    },
    bytes: Uint8Array
}
```

Notes:

- `image.bytes` holds the raw pixels, row after row, `width * bitsPerPixel / 8` bytes per row. To show them on a
  canvas, convert them to RGBA yourself.
- `image.bytes` has its own buffer. No need to free it.
- `decode` takes a complete JPEG XR file, as in a `.jxr`. A CZI tile without the JPEG XR header needs it added first.
- Invalid data throws an exception.
- Decoding only. There is no encoder.

# Sub-resolution decode

`decodeAtScale(bytes, scale)` returns the same structure, smaller.

- `scale`: 1, 2, 4, 8 or 16
- size: `ceil(width / scale)` x `ceil(height / scale)`

Speed on `samples/panel-hdr.jxr` (3440 x 1440):

| scale | size | time | faster |
|---|---|---|---|
| 1 | 3440 x 1440 | 334 ms | 1x |
| 2 | 1720 x 720 | 262 ms | 1.3x |
| 4 | 860 x 360 | 26 ms | 13x |
| 8 | 430 x 180 | 23 ms | 15x |
| 16 | 215 x 90 | 5 ms | 67x |

Colours stay within 5% of the full decode (8% at 1/16).

# Building

Only needed if you want to build `dist/jpegxr.js` yourself. It needs emscripten 5.0.7 (emsdk needs Python 3.10+):

```sh
git clone https://github.com/emscripten-core/emsdk.git ~/emsdk
~/emsdk/emsdk install 5.0.7 && ~/emsdk/emsdk activate 5.0.7
source ~/emsdk/emsdk_env.sh
make
cp wasm/jpegxr.js dist/jpegxr.js && printf '\nexport default jpegxr;\n' >> dist/jpegxr.js
shasum -a 256 dist/jpegxr.js   # c83efc7dc2b0077cffab7896bbcba40363f0b261c44e8d1f03b5074115c7818b
```

Tests, after `make`:

- `npm test`: decodes the sample at full size
- `node wasm/test-subres.js`: checks every scale

# Authors and license

Based on [bvibber/jpegxr](https://github.com/bvibber/jpegxr) by Brooke Vibber, which wraps Microsoft's jxrlib.

- jxrlib: Copyright (c) 2009, Microsoft
- JS wrapper and tweaks to the C code: Copyright (c) Brooke Vibber `<bvibber@pobox.com>`
- Sub-resolution decode and the newer-emscripten fix: Copyright (c) 2026 qinbo zhou

BSD license; see [`license.md`](license.md) and [`NOTICE.md`](NOTICE.md). Keep these notices when you redistribute.
