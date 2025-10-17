# TMG1 Format Specification (TsuMuGi format version 1)

## Overview

TMG1 is a lightweight 1-bit monochrome video data format optimized for ESP32 playback. It uses RLE + Rice coding for compactness and supports XOR-based delta frames (I/P structure). Compression is done on PC (C#), and playback is handled by ESP32.

**Target resolution: 128×64 pixels (fixed).**

---

## File Structure

```
+---------------------------+
| FileHeader (16 bytes)     |
+---------------------------+
| FrameChunk[0..N-1]        |
+---------------------------+
| [Optional] IndexChunk     |
+---------------------------+

```

### FileHeader

| Offset Field Size Description |             |   |                                         |
| ----------------------------- | ----------- | - | --------------------------------------- |
| 0                             | Magic       | 4 | "TMG1" (0x54 0x4D 0x47 0x31)            |
| 4                             | Version     | 1 | 0x01                                    |
| 5                             | Flags       | 1 | bit0: MSB-first, bit1: delta default on |
| 6                             | Width       | 2 | uint16 (fixed 128, little-endian)       |
| 8                             | Height      | 2 | uint16 (fixed 64, little-endian)        |
| 10                            | TimebaseNum | 2 | e.g. 1                                  |
| 12                            | TimebaseDen | 2 | e.g. 30 (for 30fps)                     |
| 14                            | KeyInterval | 2 | key frame interval (e.g. 60)            |

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

| Field Type Description |         |                                                     |
| ---------------------- | ------- | --------------------------------------------------- |
| FrameType              | u8      | 0 = I-frame, 1 = P-frame (XOR delta)                |
| PTSDelta               | ULEB128 | delta time from previous frame                      |
| PayloadSize            | ULEB128 | size of compressed data                             |
| FrameFlags             | u8      | bit0: has start bit per line, bit1: per-line Rice k |
| Reserved               | u8      | reserved for future use (0x00)                      |

---

## Payload (RLE + Rice encoded)

Each frame consists of 64 scanlines (fixed height). Each line contains run-lengths encoded with Rice codes.

### Line Structure

```
[LineType][(if 1) LineData]

```

* **LineType**: 1 bit (0 = unchanged, 1 = data)
* **LineData**:

  * optional StartBit (1 bit) if FrameFlags.bit0 = 1
  * optional RiceK (3 bits) if FrameFlags.bit1 = 1
  * sequence of Rice-coded run lengths (alternate 0/1 fills until width = 128)

### I-Frame

All 64 lines are encoded (LineType = 1).

### P-Frame

Only changed lines have LineType = 1; unchanged lines reuse previous frame’s data.

---

## IndexChunk (optional)

| Field Type Description |         |                         |
| ---------------------- | ------- | ----------------------- |
| Magic                  | 4B      | "TMGX"                  |
| Count                  | ULEB128 | number of frames        |
| Offset[i]              | uint64  | file offset to frame[i] |

---

## ESP32 Decoding Summary

1. Read FileHeader.
2. For each frame:

   * Read FrameHeader.
   * For each line (64 total):

     * If P-frame and LineType=0 → copy previous line.
     * Otherwise decode Rice runs and rebuild pixels (width 128).
   * XOR with previous frame if P-frame.
   * Display frame.

---

## Recommended Parameters

| Field Recommended Value |                                |
| ----------------------- | ------------------------------ |
| Bit order               | MSB-first                      |
| Width / Height          | 128×64 (fixed)                 |
| KeyInterval             | 60                             |
| Rice k                  | 1 (default), per-line optional |
| StartBit                | 0 default                      |

---

## Notes

* **Reserved** byte in FrameHeader is kept for future extensions (CRC, compression mode flags, etc.)
* Each frame is line-independent for robust error recovery.
* Typical 128×64 frame: ~2–250 bytes depending on motion.
