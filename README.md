# Firmware Image Toolkit

A validated, sparse-address firmware image model with Intel HEX, Motorola S-record, and raw binary conversion.

The toolkit verifies record checksums and lengths, normalizes segments, detects conflicting overlaps, preserves entry points, fills sparse gaps intentionally, applies auditable patches, computes stable hashes, and produces byte-level semantic diffs. It forms a safe input layer for flashers, bootloaders, release pipelines, and firmware inspectors.

## Quick start

```bash
npm install
npm test
node src/cli.js info examples/blink.hex --from ihex
node src/cli.js convert examples/blink.hex --from ihex --to srec --output blink.s37
node src/cli.js convert examples/blink.hex --from ihex --to raw --output blink.bin
```

```js
import {
  parseImage, analyzeImage, patchImage, diffImages,
  toIntelHex, toSRecord, toRaw
} from '@opencircuitlabs/firmware-image-toolkit';

const image = parseImage(hexText, 'ihex');
const info = analyzeImage(image);
const patched = patchImage(image, 0x08001000, calibrationBytes);
const changes = diffImages(image, patched);
```

## Image model

An image contains sorted, nonempty `{address, data}` segments and metadata. Normalization:

- accepts 32-bit unsigned addresses;
- merges adjacent segments;
- merges byte-identical overlaps;
- rejects conflicting overlaps unless an explicit patch operation is used;
- rejects segments that exceed the address space.

Sparse images remain sparse. Raw conversion requires a bounded output span and fills gaps with `0xFF` by default, preventing accidental multi-gigabyte allocations.

## Intel HEX

The parser verifies every checksum and record length. It supports data, EOF, extended segment/linear addressing, and segment/linear entry-point records. Unknown records are retained as warnings. Generation splits data at both configured record widths and 64 KiB address boundaries, emitting deterministic uppercase records.

## Motorola S-record

The parser handles S1/S2/S3 data, S5/S6 record counts, and S7/S8/S9 termination/entry points with checksum validation. Generation uses S3 data for full 32-bit addressing, emits a header and count record, and terminates with S7.

## Analysis and diffs

`analyzeImage` reports segment/content bytes, occupied span, holes, entry point, and a stable SHA-256 that includes addresses as well as data. `diffImages` groups consecutive changed addresses and distinguishes absent bytes with `null`, suitable for review or delta packaging.

## License

MIT
