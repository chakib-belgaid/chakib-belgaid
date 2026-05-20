# Chakib Belgaid, PhD

I build AI systems, developer tools, creative media software, and energy-aware infrastructure.

The thread through my work is measurement and usable experimentation: LLM agents should be evaluated, production systems should be observable, creative tools should be tangible, and energy claims should be reproducible before they are optimized.

## Work

My current focus is the overlap between:

- LLM systems: tool-calling agents, retrieval, structured outputs, and evaluation loops.
- Backend systems: APIs, software architecture, automation, and production-facing developer tools.
- Creative tools: generative interfaces, audiobooks, and small web apps that are useful as tools, not only demos.
- Energy-aware software: measurement, benchmarking, profiling overhead, reproducibility, and regression detection.
- Research prototypes that become usable tools, not only demos.

## Current Project: Wattch

**[Wattch](https://github.com/chakib-belgaid/Wattch)** is lightweight energy-profiling infrastructure for developers and AI coding agents.

It is a Rust daemon and CLI for low-overhead energy measurement, built around reproducible benchmarks and machine-readable reports. The long-term direction is IDE and MCP integration, so AI agents can detect energy regressions, reason about measurement confidence, and suggest greener code changes without hiding overhead or uncertainty.

Status: work in progress.

## Recent Work

Recent public activity in May 2026 includes:

- **[Fractal Brushes](https://github.com/chakib-belgaid/fractal-brushes)** - a generative-art web app for fractal and symmetry-based backgrounds. [Open the live demo](https://chakib-belgaid.github.io/fractal-brushes/).
- **[Whispbook](https://github.com/chakib-belgaid/whispbook)** - a self-hosted audiobook studio for turning selectable-text documents into chaptered audio with subtitles.

## Selected Repositories

### Energy-aware software

- **[jreferral](https://github.com/chakib-belgaid/jreferral)** - recommends energy-efficient JVM configurations for Java software.
- **[IJoules](https://github.com/chakib-belgaid/IJoules)** - measures energy consumption of Python code on macOS / Intel CPU.

### AI / developer tools

- **[Wattch](https://github.com/chakib-belgaid/Wattch)** - current project; Rust daemon and CLI for developer-facing energy profiling.
- **[whispbook](https://github.com/chakib-belgaid/whispbook)** - TypeScript audiobook tooling for document-to-audio workflows.

### Creative web / media tools

- **[Fractal Brushes](https://github.com/chakib-belgaid/fractal-brushes)** - static frontend for generating artsy fractal and symmetry backgrounds. [Live demo](https://chakib-belgaid.github.io/fractal-brushes/).

## Research Foundation

![PhD thesis visual map](assets/thesis-brain-map-ai.png)

My PhD work focused on energy-aware software engineering: measurement, benchmarking, testing, optimization, language/runtime behavior, and reproducibility. Wattch builds on that foundation by turning the research concerns into developer infrastructure: repeatable runs, explicit overhead, structured reports, and tooling that can fit into an engineering workflow.

- **[chakib_belgaid_thesis](https://github.com/chakib-belgaid/chakib_belgaid_thesis)** - thesis source and materials.

Source: [thesisBrainMap.svg](assets/thesisBrainMap.svg) · [thesisBrainMap.drawio](assets/thesisBrainMap.drawio)

## Engineering Taste

- Measurable before optimized.
- Local-first when possible.
- Explicit about uncertainty and overhead.
- Useful to developers, not only impressive in demos.

## Contact

[Email](mailto:chakib.belgaid@gmail.com) · [LinkedIn](https://linkedin.com/in/chakib-belgaid) · [Blog](https://chakib-belgaid.github.io)

## Mobile TTS Streaming Blueprint

I added a practical implementation blueprint for lightweight mobile chapter narration with **MeloTTS**, **CosyVoice 2**, and **QweTTS**, including:

- ONNX/TFLite model format guidance,
- chunked streaming to avoid freezes on large files,
- ring-buffer/backpressure architecture,
- long-file stress test criteria,
- optional phrase highlighting using alignment timelines.

See: [`docs/mobile-tts-streaming.md`](docs/mobile-tts-streaming.md).
