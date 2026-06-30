# TMG1 Format Specification

**English** | [日本語](tmg1-format.ja.md)

> Status: draft. The format is still evolving; only the current version
> (`version = 2`) is documented and supported. No backward compatibility with
> earlier, unpublished revisions is maintained.

## Overview

TMG1 is a lightweight 1-bit monochrome (bitplane) video format optimized for
ESP32 playback. It combines run-length and entropy coding for compactness and
supports XOR-based delta frames (I/P structure). Compression is performed on a
PC by the Rust CLI ([`tmg1-cli`](https://github.com/tmg1-labs/tmg1-cli)) on top
of the shared C++ codec library
([`tmg1-codec`](https://github.com/tmg1-labs/tmg1-codec)); playback is handled
on the ESP32 by [`tmg1-esp32-demo`](https://github.com/tmg1-labs/tmg1-esp32-demo).

**Recommended target resolution: 128×64 pixels.**

All multi-byte integers are little-endian unless noted. `ULEB128` denotes an
unsigned LEB128 variable-length integer.

---

## File Structure

```
+---------------------------+
| FileHeader (16 bytes)     |
+---------------------------+
| FrameChunk[0 .. N-1]      |
+---------------------------+
| IndexChunk (optional)     |
+---------------------------+
```

### FileHeader (16 bytes)

| Offset | Field        | Size | Description                                                          |
| ------ | ------------ | ---- | -------------------------------------------------------------------- |
| 0      | Magic        | 4    | `"TMG1"` (0x54 0x4D 0x47 0x31)                                       |
| 4      | Version      | 1    | `0x02`                                                               |
| 5      | Flags        | 1    | bit0: MSB-first, bit1: reserved (unused), bit2: Range coder enabled  |
| 6      | Width        | 2    | uint16                                                               |
| 8      | Height       | 2    | uint16                                                               |
| 10     | TimebaseNum  | 2    | e.g. `1`                                                             |
| 12     | TimebaseDen  | 2    | e.g. `30` (for 30 fps)                                               |
| 14     | KeyInterval  | 2    | key frame interval (e.g. `60`)                                       |

---

## Frame Chunk

```
+------------------------+
| FrameHeader (variable) |
+------------------------+
| Payload (variable)     |
+------------------------+
```

### FrameHeader

| Field            | Type    | Description                                                                                           |
| ---------------- | ------- | ---------------------------------------------------------------------------------------------------- |
| FrameType        | u8      | `0` = I-frame, `1` = P-frame (XOR delta)                                                              |
| PTSDelta         | ULEB128 | delta time from the previous frame                                                                    |
| PayloadSize      | ULEB128 | size of the compressed payload in bytes                                                               |
| FrameFlags       | u8      | bit0: per-line start bit, bit1: per-line Rice k, bit2: per-frame Rice k (bit1 and bit2 are exclusive) |
| PredictionMethod | u8      | `0`: None, `1`: Left, `2`: Up                                                                         |

---

## Payload

The entropy coder used for the payload is selected by `FileHeader.Flags.bit2`.

### Rice Coder Payload (default)

When the Range coder is disabled, the payload is compressed with Run-Length
Encoding (RLE) followed by Golomb-Rice coding. A frame is a sequence of
scanlines; each data line contains Rice-coded run lengths.

#### Line Structure

```
[LineType][(if 1) LineData]
```

- **LineType**: 1 bit (`0` = unchanged/empty, `1` = data)
- **LineData**:
  - optional StartBit (1 bit) if `FrameFlags.bit0 = 1`
  - optional RiceK (3 bits) if `FrameFlags.bit1 = 1` (per-line mode)
  - a sequence of Rice-coded run lengths (alternating 0/1 fills until `width`)

#### Per-frame mode

If `FrameFlags.bit2` is set, a single Rice `k` precedes the lines and the
individual per-line RiceK is omitted:

```
[FrameRiceK (3 bits)][Line ...][Line ...] ...
```

### Range Coder Payload

When `FileHeader.Flags.bit2` is set, the payload is **not** RLE-encoded.
Instead, the bitplane data (after prediction filtering) is compressed line by
line by a Range coder: each line starts with a 1-bit line type (`1` = has data,
`0` = empty line, skipped), and for data lines the `width` pixel bits follow in
order. The empty-line skip is shared with the Rice path.

The Range coder is a form of arithmetic coding that achieves higher compression
than Golomb-Rice when bit probabilities are far from 50%, at a higher
computational cost.

#### Technical details

- **Adaptive frequency model with context** — an adaptive `FrequencyModel`
  estimates the probability of the next bit. A first-order context model selects
  the distribution by the value of the preceding bit (two distributions: for a
  previous `0` and for a previous `1`). Each starts uniform (50/50); only the
  active context is updated per bit; counts are periodically rescaled (halved)
  to bound them and adapt to changing statistics.
- **Coding state** — the coder maintains a numeric range as two 64-bit integers:
  `low` (bottom of the range) and `range` (its width). Encoding a bit narrows
  the range by the modeled probability; `low` gradually forms the bitstream.
- **Normalization** — as `range` shrinks, the most significant bytes of `low`
  are shifted out and `range` is shifted left to restore magnitude. The decoder
  performs the inverse, shifting bytes in to stay synchronized.

### I-frame

- **Rice coder**: only lines containing data are encoded (LineType = 1);
  empty (all-zero) lines are still marked LineType = 0.
- **Range coder**: the entire bitplane is fed to the coder (with the same
  per-line empty-line skip).

### P-frame

- **Rice coder**: only changed lines have LineType = 1; unchanged lines reuse
  the previous frame's data.
- **Range coder**: the XOR delta of the entire bitplane is fed to the coder.
  Identical frames are skipped via VFR, but partial updates are not supported —
  when the Range coder is active, a P-frame always carries the full delta.

---

## IndexChunk (optional)

Appended at end-of-file to support seeking.

| Field     | Type    | Description              |
| --------- | ------- | ------------------------ |
| Magic     | 4 B     | `"TMGX"`                 |
| Count     | ULEB128 | number of frames         |
| Offset[i] | uint64  | file offset to frame `i` |

---

## Prediction Filtering

Before RLE + Rice (or Range) coding, TMG1 can apply a prediction filter to the
frame data (or, for P-frames, the delta data). The encoder picks the method that
minimizes the compressed payload per frame, and records it in
`FrameHeader.PredictionMethod`.

| ID  | Method | Forward filter                          | Inverse                                 |
| --- | ------ | --------------------------------------- | --------------------------------------- |
| `0` | None   | —                                       | —                                       |
| `1` | Left   | `filtered[x] = raw[x] ^ raw[x-1]` (byte) | `raw[x] = filtered[x] ^ raw[x-1]`      |
| `2` | Up     | `filtered[y] = raw[y] ^ raw[y-1]` (byte) | `raw[y] = filtered[y] ^ raw[y-1]`      |

The encoder tries all three methods, compares payload sizes, and writes the
winning ID. The decoder reads the ID and applies the matching inverse filter
after decompression.

---

## ESP32 Decoding Summary

1. Read FileHeader.
2. For each frame:
   - Read FrameHeader.
   - For each line: if P-frame and LineType = 0, copy the previous line;
     otherwise decode and rebuild pixels.
   - XOR with the previous frame if P-frame.
   - Apply the inverse prediction filter.
   - Display the frame.

---

## Recommended Parameters

| Field          | Recommended value              |
| -------------- | ------------------------------ |
| Bit order      | MSB-first                      |
| Width × Height | 128 × 64                       |
| KeyInterval    | 60                             |
| Rice k         | 1 (default), per-line optional |
| StartBit       | 0 (default)                    |

---

## Notes

- Each frame is line-independent for robust error recovery.
- A typical 128×64 frame is ~2–250 bytes depending on motion.
