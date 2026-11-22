# jpegxr-js - JPEG XR decoding in JavaScript, with fast sub-resolution decode

jpegxr-js is an independent continuation of [bvibber/jpegxr](https://github.com/bvibber/jpegxr) by Brooke Vibber:
Rust and JavaScript (WebAssembly) wrappers around the C libjpegxr / jxrlib codec open-sourced by Microsoft. This
repository starts from bvibber/jpegxr at commit `3e8b298` with the changes below (2026-05-06); later changes of the
original project are not included, and its history is in its own repository. Only the JavaScript part is kept: the
Rust crate of the original project is not included. This version is not endorsed by the original authors.

What this version adds:

- **Sub-resolution decode in the JS API**: `codec.decodeAtScale(bytes, scale)`, with `scale` 1, 2, 4, 8 or 16 (see
  "Sub-resolution decode" below). jxrlib's inverse transform skips the high-frequency subbands, so a 1/4 or 1/8
  image comes out about 10 times faster than a full decode followed by a canvas downscale, with average colour
  within about 3% of the full image. Useful for thumbnails and fit-to-window views of large images.
- Works with newer emscripten output (`Module.ready` is no longer there; the methods are attached to the module
  directly). The original `decode()` API is unchanged.

This code is already in use in a real project.

Only decoding is supported.

Please report problems with this version here.

# Authors

The wrapped C JPEG XR library was written by many fine folks at Microsoft!
Rust and JS code wrapping it, and tweaks to the C code, are by Brooke Vibber `<bvibber@pobox.com>`.
Sub-resolution decode in the JS API and the newer-emscripten fix are by qinbo zhou.

# License and copyright

BSD-style license; see `license.md`, `NOTICE.md` and the headers in source files.

- jxrlib: Copyright (c) 2009, Microsoft
- Rust and JS wrappers: Copyright (c) Brooke Vibber
- Changes in jpegxr-js: Copyright (c) 2026 qinbo zhou

All are under the same BSD terms; keep these notices when you redistribute.

# Usage (JS)

No npm package is needed. Copy the prebuilt [`dist/jpegxr.js`](dist/jpegxr.js) into your project: one file with
the WebAssembly binary inlined (185 KB, no separate `.wasm` to serve), already an ES module, the same file that is in
real use (see [`dist/README.md`](dist/README.md) for how it was built and its checksum). Or build it yourself (see
"Building (JS)" below): `make` writes `wasm/jpegxr.js`; add `export default jpegxr;` at its end to use it as an ES
module.

Then (browsers, Vite and other bundlers):

```js
import jpegxr from './jpegxr.js';

const codec = await jpegxr();
const image = codec.decode(bytes);              // bytes: Uint8Array of a .jxr file
const quarter = codec.decodeAtScale(bytes, 4);  // the same image at 1/4 size, much faster
```

It also loads lazily, for example in a Web Worker: `const jpegxr = (await import('./jpegxr.js')).default;`.

The module exports a factory function which asynchronously prepares the WebAssembly modules and returns an API wrapper object via a `Promise`.

Call its `decode` method with a `Uint8Array` of input bytes to get back an object with the following structure:

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

Exceptions may be thrown in case of invalid data. The `bytes` array is standalone and backed by its own buffer, and does not need to be manually freed -- however this all incurs the cost of a single copy of both input and output data in/out of the WebAssembly module.

## Sub-resolution decode

`decodeAtScale(bytes, scale)` returns the same structure as `decode`, at `ceil(width / scale)` by
`ceil(height / scale)` pixels. `scale` is 1, 2, 4, 8 or 16; 1 is the same as `decode`.

```js
jpegxr().then((codec) => {
    let full = codec.decode(bytes);               // 3440 x 1440
    let quarter = codec.decodeAtScale(bytes, 4);  //  860 x  360, about 12x faster
});
```

Measured on the bundled HDR sample `samples/panel-hdr.jxr` (3440 x 1440), average colour compared with the full
decode:

| scale | size | time | average R / G / B vs full |
|---|---|---|---|
| 1 | 3440 x 1440 | 334 ms | 1.000 / 1.000 / 1.000 |
| 2 | 1720 x 720 | 262 ms | 1.036 / 1.038 / 1.043 |
| 4 | 860 x 360 | 26 ms | 0.973 / 0.984 / 0.990 |
| 8 | 430 x 180 | 23 ms | 0.970 / 0.982 / 0.989 |
| 16 | 215 x 90 | 5 ms | 0.921 / 0.951 / 0.969 |

In C, the same is available as `image_decode_set_thumbnail_dims(decode, w, h)` before `image_decode_copy_all`
(pass 0, 0 for full resolution).


# Features

Currently sports the ability to read basic image format (width/height/pixel format) from a JPEG XR image and decode its data to memory.

HDR images with 32-bit floating point RGBA elements, as saved from the NVIDIA game screen capture tool, appear to decode correctly.


# Building (JS)

Building the JS via emscripten should work on macOS and Linux.

Install the [emscripten SDK](https://emscripten.org/) and set it up in `PATH`. Either run `make` directly, or `npm run-script build` which runs `make` itself.

To reproduce `dist/jpegxr.js` exactly, use emscripten 5.0.7 (emsdk needs Python 3.10 or newer):

```sh
git clone https://github.com/emscripten-core/emsdk.git ~/emsdk
~/emsdk/emsdk install 5.0.7 && ~/emsdk/emsdk activate 5.0.7
source ~/emsdk/emsdk_env.sh
make
cp wasm/jpegxr.js dist/jpegxr.js && printf '\nexport default jpegxr;\n' >> dist/jpegxr.js
shasum -a 256 dist/jpegxr.js   # c83efc7dc2b0077cffab7896bbcba40363f0b261c44e8d1f03b5074115c7818b
```

This gives a byte-identical file (checked on macOS, Apple Silicon, 2026-11-22).

`node wasm/test-subres.js` checks the sub-resolution decode on the same sample at every scale (sizes, timing and
average colour).

Run `npm test` to run a verification script which loads a sample floating point HDR screenshot and calculates average red, green, and blue intensities to prove it loaded correctly. It prints the image info and ends like this, with no exceptions thrown:

```
average red brightness: 1.4860534716607372
average green brightness: 2.4572176722254246
average blue brightness: 3.867482314063597
```

