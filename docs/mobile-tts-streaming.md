# Mobile Streaming TTS Architecture (MeloTTS, CosyVoice 2, QweTTS)

This document defines a lightweight, mobile-first architecture to support chapter-length streaming narration using three model families:

- **MeloTTS** (ONNX)
- **CosyVoice 2** (ONNX/TFLite converted acoustic + vocoder stack)
- **QweTTS** (ONNX)

The design prioritizes:

1. low-memory runtime on mobile,
2. steady playback without freezes on long chapters,
3. bounded latency for chunk-to-audio conversion,
4. optional phrase-level highlighting when timestamps are available.

---

## 1) Runtime Targets

### Model format policy

- Prefer **INT8 quantized ONNX** for compatibility with `onnxruntime-mobile` and NNAPI/CoreML delegates.
- Use **TFLite INT8/FP16** if the vendor model ships with better mobile kernels.
- Keep per-model bundles modular:
  - `acoustic_model.(onnx|tflite)`
  - `vocoder.(onnx|tflite)`
  - `tokenizer.json` / `lexicon.txt`
  - `config.json` (sample rate, hop length, voices, normalization)

### Performance budget (recommended)

- Cold start < 1.5 s on mid-range device.
- Steady-state synthesis real-time factor (RTF) <= 0.8 (faster than playback).
- Peak memory for one active voice <= 220 MB.
- Chunk synthesis deadline <= 300 ms (for short phrase chunks).

---

## 2) End-to-end streaming pipeline

```text
Chapter text
  -> sentence/phrase segmenter
  -> chunk scheduler (adaptive size)
  -> TTS inference worker pool (size=1..2)
  -> audio ring buffer (PCM)
  -> player output stream
  -> optional subtitle/highlight timeline
```

### Core idea

Instead of generating full chapter audio, we synthesize and enqueue **small chunks** continuously while playback is ongoing.

- Playback starts once a minimum prebuffer is ready (e.g. 1.0–1.5 s).
- Synth continues ahead of playback by target lead time (e.g. 8–15 s).
- Backpressure prevents over-generation and memory spikes.

---

## 3) Chunking strategy to prevent freezes

### Segmentation rules

1. Split by strong punctuation (`.`, `!`, `?`) first.
2. Then split long sentences by commas/semicolons.
3. Cap chunk length by characters and token count:
   - `max_chars = 180`
   - `max_tokens = 80`
4. Merge ultra-short chunks with neighbors (minimum 35 chars).

### Adaptive chunking

If runtime falls behind (buffer lead < 3 s):

- reduce chunk size by 25–40%,
- lower vocoder quality preset if available,
- increase synthesis priority thread.

If runtime stays comfortably ahead (buffer lead > 20 s):

- increase chunk size to reduce scheduler overhead.

This adaptive loop avoids hard stalls in very large chapters.

---

## 4) Streaming buffer design

### Audio ring buffer

- Store PCM16 frames in a lock-free ring buffer.
- Capacity target: 20–30 s audio.
- Watermarks:
  - `low = 2.0 s` (urgent synth)
  - `target = 10.0 s`
  - `high = 25.0 s` (pause synth)

### Threading model

- **UI thread**: controls play/pause/seek and highlighting updates.
- **Synthesis thread**: runs model inference chunk-by-chunk.
- **Audio callback thread**: pulls PCM from ring buffer (must never block).

Avoid any model inference on the audio callback thread.

---

## 5) Model-specific notes

## MeloTTS (light path)

- Export acoustic + vocoder to ONNX with dynamic sequence length.
- Use INT8 quantization where quality remains acceptable.
- Cache speaker embedding and normalization pipeline.

## CosyVoice 2

- If full stack is heavy, split into:
  - lightweight acoustic backbone,
  - reduced-bandwidth vocoder profile for mobile.
- Use ONNX EP fallback order:
  1. NNAPI/CoreML
  2. XNNPACK/CPU

## QweTTS

- Use reduced decoder depth preset if model family supports it.
- Quantize linear layers and keep sensitive output layers in FP16 when needed.

---

## 6) Phrase highlighting (when supported)

Highlighting requires per-phoneme/per-token duration or alignment.

### If model emits alignments/durations

- Build timeline entries:
  - `chunk_id`
  - `text_start`, `text_end`
  - `audio_start_ms`, `audio_end_ms`
- During playback, map current playback timestamp to text span.
- Update highlighted phrase on UI every 50–100 ms.

### If model does not emit alignment

Fallback options:

1. Use heuristic proportional timing by token count.
2. Run lightweight forced-alignment post-pass in background.
3. Disable fine phrase highlight and keep sentence-level highlight.

Recommendation: expose a capability flag per voice:

```json
{
  "supports_alignment": true,
  "highlight_granularity": "phrase"
}
```

---

## 7) Freeze and long-file testing plan

## Stress scenarios

1. **Short chapter**: 2k words.
2. **Medium chapter**: 12k words.
3. **Large chapter**: 40k+ words.
4. Background CPU pressure (simulated load).
5. Device thermal throttling session (20+ minutes).

### Metrics to capture

- Buffer underruns (count + timestamp).
- Chunk synthesis latency (p50/p95/p99).
- RTF over time.
- Peak RSS memory.
- GC pauses (if managed runtime wrapper).
- Audio callback missed deadlines.

### Pass criteria

- Zero audible gaps for medium chapter.
- <= 1 recoverable micro-gap for large chapter under load.
- No app ANR / watchdog termination.

---

## 8) Minimal integration interfaces

```ts
export interface TtsChunk {
  id: string;
  text: string;
  tokens?: number;
}

export interface SynthResult {
  chunkId: string;
  pcm16: Int16Array;
  sampleRate: number;
  timeline?: Array<{ startMs: number; endMs: number; textStart: number; textEnd: number }>;
}

export interface StreamingTtsEngine {
  load(modelId: "melotts" | "cosyvoice2" | "qwetts"): Promise<void>;
  start(chapterText: string): Promise<void>;
  pause(): void;
  resume(): void;
  stop(): void;
  seek(ms: number): Promise<void>;
  onHighlight(cb: (textStart: number, textEnd: number) => void): void;
}
```

Note: Replace `"qwetts"` with canonical id used in your codebase (e.g. `"qwetts"`).

---

## 9) Recommended default values

- Prebuffer before play: **1200 ms**
- Target lead buffer: **10 s**
- Max lead buffer: **25 s**
- Initial chunk size: **120 chars**
- Emergency chunk size: **70 chars**
- Highlight update tick: **75 ms**

These defaults are conservative for smooth playback on mid-tier mobile hardware.
