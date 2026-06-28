# TMG1 Format Specification (TsuMuGi format version 1)

## Overview

TMG1 is a lightweight 1-bit monochrome video data format optimized for ESP32 playback. It uses RLE + Rice coding for compactness and supports XOR-based delta frames (I/P structure). Compression is done on PC by the Rust CLI (`tmg1-cli`) on top of the shared C++ codec library (`tmg1-codec`), and playback is handled by ESP32. (The former .NET/C# encoder is discontinued.)

**Target resolution: 128×64 pixels (recommended).**

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
| 4                             | Version     | 1 | 0x02                                    |
| 5                             | Flags       | 1 | bit0: MSB-first, bit1: reserved (unused), bit2: Range Coder enabled |
| 6                             | Width       | 2 | uint16 (little-endian)       |
| 8                             | Height      | 2 | uint16 (little-endian)        |
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
| FrameFlags             | u8      | bit0: has start bit per line, bit1: per-line Rice k, bit2: per-frame Rice k (bit1 and bit2 are mutually exclusive) |
| PredictionMethod       | u8      | `0`: None, `1`: Left, `2`: Up                      |

---

## Payload (RLE + Entropy encoded)

The method used for entropy encoding is determined by the `Flags` field in the `FileHeader`.

### Rice Coder Payload (Default)

When Range Coder is disabled, the payload is compressed using Run-Length Encoding (RLE) followed by Golomb-Rice coding. Each frame consists of multi scanlines, and each line contains Rice-coded run lengths.

#### Line Structure

```
[LineType][(if 1) LineData]

```

* **LineType**: 1 bit (0 = unchanged, 1 = data)
* **LineData**:

  * optional StartBit (1 bit) if FrameFlags.bit0 = 1
  * optional RiceK (3 bits) if FrameFlags.bit1 = 1 (per-line mode)
  * sequence of Rice-coded run lengths (alternate 0/1 fills until width)

#### Frame-level Data (Per-frame mode)

If `FrameFlags.bit2` is set, the payload has a slightly different structure:

```
[FrameRiceK][Line...][Line...]...
```

* **FrameRiceK**: A single 3-bit Rice `k` parameter for the entire frame.
* **LineData**: In this mode, lines do not contain individual `RiceK` values.

### Range Coder Payload

When `FileHeader.Flags.bit2` is set, the payload is **not** RLE-encoded. Instead, the bitplane data (after prediction filtering) is compressed line by line using a **Range Coder**: each line begins with a 1-bit line type (1 = has data, 0 = empty line, skipped), and for data lines the `width` pixel bits are then coded in order. The empty-line skip is shared with the Rice path.

The Range Coder is a form of arithmetic coding that can achieve higher compression ratios than Golomb-Rice coding, especially for data where the probability of bits is not close to 50%, at the cost of higher computational complexity.

#### Technical Details

- **Adaptive Frequency Model with Context**: The coder uses an adaptive frequency model (`FrequencyModel`) to dynamically estimate the probability of the next bit being a 0 or a 1. To further improve compression, it employs a **first-order context model**. This means the probability distribution for the current bit is chosen based on the value of the preceding bit.
  - The model maintains two separate frequency distributions: one for when the previous bit was `0`, and one for when it was `1`.
  - Each model is initialized with a uniform distribution (50/50 probability).
  - After encoding or decoding a bit, only the frequency counts corresponding to the correct context are updated.
  - To prevent overflow and adapt to changing data statistics, the frequency counts are periodically rescaled (halved) when a total count threshold is reached.

- **Coding Process**: The core of the coder maintains a numerical range defined by two 64-bit integers:
  - `low`: The bottom of the current range.
  - `range`: The width of the current range.

  To encode a bit, this range is narrowed based on the bit's probability, as estimated by the frequency model. The `low` value gradually forms the compressed bitstream.

- **Normalization**: As the `range` becomes smaller, its precision decreases. To counteract this, the coder performs normalization. When the `range` falls below a certain threshold, the most significant bytes of `low` are shifted out to the compressed stream, and the `range` is shifted left to restore its magnitude. The decoder performs the inverse operation, shifting in bytes from the stream to maintain synchronization.

### I-Frame

- **Rice Coder**: Only lines that contain data are encoded (LineType = 1); empty (all-zero) lines are still marked LineType = 0 even in I-frames.
- **Range Coder**: The entire bitplane is fed to the Range Coder (with the same per-line empty-line skip).

### P-Frame

- **Rice Coder**: Only changed lines have LineType = 1; unchanged lines reuse previous frame’s data.
- **Range Coder**: The XOR delta data of the entire bitplane is fed to the Range Coder. Unchanged frames are still skipped via VFR, but partial updates are not supported. If Range Coder is active, P-frames contain the full delta.


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
   * For each line:
     * If P-frame and LineType=0 → copy previous line.
     * Otherwise decode Rice runs and rebuild pixels.
   * XOR with previous frame if P-frame.
   * Display frame.

---

## Recommended Parameters

| Field Recommended Value |                                |
| ----------------------- | ------------------------------ |
| Bit order               | MSB-first                      |
| Width / Height          | 128×64                         |
| KeyInterval             | 60                             |
| Rice k                  | 1 (default), per-line optional |
| StartBit                | 0 default                      |

---

## Prediction Filtering

To improve compression efficiency, TMG1 can apply a prediction filter to the frame data (or delta data for P-frames) before it is compressed with RLE + Rice coding. The encoder dynamically selects the best prediction method for each frame to minimize the size of the compressed payload.

The selected method is stored in the `PredictionMethod` field of the `FrameHeader`.

### Prediction Methods

| ID  | Method | Description                                      |
| --- | ------ | ------------------------------------------------ |
| `0` | None   | No filter is applied.                            |
| `1` | Left   | `filtered[x] = raw[x] ^ raw[x-1]` (byte-wise)    |
| `2` | Up     | `filtered[y] = raw[y] ^ raw[y-1]` (byte-wise)    |

### Encoder Behavior

- For each frame, the encoder tries compressing the data with all three prediction methods (None, Left, Up).
- It then compares the resulting payload sizes and selects the method that yields the smallest size.
- The ID of the chosen method is written to the `PredictionMethod` field in the frame header.

### Decoder Behavior

- The decoder must read the `PredictionMethod` field from the frame header.
- After decompressing the payload to get the filtered data, it must apply the corresponding inverse filter to reconstruct the original pixel data.
  - For `Left`, the inverse operation is `raw[x] = filtered[x] ^ raw[x-1]`.
  - For `Up`, the inverse operation is `raw[y] = filtered[y] ^ raw[y-1]`.

---

## Notes

* Each frame is line-independent for robust error recovery.
* Typical 128×64 frame: ~2–250 bytes depending on motion.
