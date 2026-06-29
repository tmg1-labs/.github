# TMG1 Labs

**TMG1** is a lightweight 1-bit monochrome (bitplane) video format designed for
microcontrollers — small enough to store and play back smooth animation on an
ESP32 driving a 128×64 OLED.

Compression runs on a PC; playback runs on the device. A single shared C++
codec is used on both sides so the encoder and decoder never drift apart.

## Repositories

| Repository | Language | Role |
| ---------- | -------- | ---- |
| [**tmg1-codec**](https://github.com/tmg1-labs/tmg1-codec) | C++17 | Shared codec library (encoder/decoder). Used by both the firmware and the CLI via a C FFI. |
| [**tmg1-arduino**](https://github.com/tmg1-labs/tmg1-arduino) | C++ / PlatformIO | ESP32 firmware: reads a `.tmg1` file from flash and plays it on a U8g2 OLED. |
| [**tmg1-cli**](https://github.com/tmg1-labs/tmg1-cli) | Rust | Command-line encoder/transcoder (ffmpeg pipeline → `.tmg1`). |

## Format Specification

- [TMG1 Format Specification (English)](docs/tmg1-format.md)
- [TMG1 フォーマット仕様書（日本語）](docs/tmg1-format.ja.md)

## Highlights

- **64-bit Range coder** — `__uint128_t`-free arithmetic coding (works on MSVC).
- **Rice coder** — run-length entropy coding with fixed / per-line / per-frame `k`.
- **Prediction filters** — None / Left / Up, auto-selected per frame.
- **Delta (P) frames**, scene-change detection, and variable frame rate.
- **Streamed, low-memory I/O** — frames are processed one at a time.

## License

Each repository is published under the MIT License.
</content>
