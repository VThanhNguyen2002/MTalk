# Mtalk — Translate for Drivers

[![Android](https://img.shields.io/badge/Platform-Android_10%2B_(API_29%2B)-3DDC84.svg?style=flat&logo=android)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Language-Kotlin-7F52FF.svg?style=flat&logo=kotlin)](https://kotlinlang.org)
[![Compose](https://img.shields.io/badge/UI-Jetpack_Compose-4285F4.svg?style=flat&logo=jetpackcompose)](https://developer.android.com/jetpack/compose)
[![Status](https://img.shields.io/badge/Status-Foundation_Phase-orange.svg)]()

> **A real-world, high-discipline voice translation application engineered specifically for taxi and delivery drivers communicating with foreign passengers and customers.**

---

## The Real Problem

Drivers often encounter foreign passengers and delivery customers where communication must be immediate, precise, and low-friction. Critical interactions include:
- Confirming destinations, landmarks, and route adjustments ("Đi thẳng", "Rẽ trái", "Rẽ phải").
- Clarifying payment methods, cash exchange, and tips ("Tiền mặt hay chuyển khoản?", "Tôi không có tiền lẻ").
- Arrival updates and waiting limits ("Tôi đến rồi", "Đợi tôi 5 phút", "Tôi đang bị kẹt xe").
- Delivery gate drop-off and lobby coordination ("Bạn có thể xuống đây không?").

General-purpose translation apps fail in this context: they are cluttered with complex menus, assume quiet environments, require multi-step navigation, and rely heavily on uninterrupted cellular data.

**Mtalk** is designed around the high-noise, high-urgency reality of vehicle cabins and delivery operations.

---

## Core Architecture & Conceptual Pipeline

```
  Driver (Vietnamese)                             Passenger (Foreign Language)
          │                                                    │
          ▼                                                    ▼
    [ Audio Input ]                                      [ Audio Input ]
          │                                                    │
          ▼                                                    ▼
 [ Speech-to-Text (STT) ]                            [ Speech-to-Text (STT) ]
          │                                                    │
          ▼                                                    ▼
    [ Translation ]                                      [ Translation ]
          │                                                    │
          ▼                                                    ▼
   [ Translated Text ]                                  [ Translated Text ]
          │                                                    │
          ▼                                                    ▼
 [ Text-to-Speech (TTS) ]                            [ Text-to-Speech (TTS) ]
```

### Key Architectural Tenets
1. **Dual Output (Audio + Visual)**: Voice output is supplemented with high-contrast, large text on-screen so that when engine or traffic noise drowns out the phone speaker, the message remains clear.
2. **Provider-Agnostic Core**: Business logic and UI depend only on abstract domain contracts (`SpeechProvider`, `TranslationProvider`, `TtsProvider`, `AudioSource`). Engines can be swapped or hybridized without refactoring the UI.
3. **Decoupled Audio Source**: The audio pipeline supports an interface-based abstraction for audio input (e.g., conceptual hardware capture vs. deterministic synthetic WAV/PCM replay) so headless CI does not require physical microphone hardware. Concrete classes will remain proportional to evidence.
4. **Driver Phrasebook**: Instant, zero-latency, offline-capable triggers for the top 80% of repetitive operational phrases.

---

## Project Status: Foundation Phase

Implementation of application features has **not** started. In accordance with disciplined software engineering, we have established the formal engineering foundation, security policies, verification matrix, and decision tracking before writing application code.

### Architectural Decisions Classification

- **Established Requirements**:
  - Minimum compatibility: Android 10 (`minSdk = 29`).
  - Native UI framework: Jetpack Compose with Kotlin.
  - Distribution: Direct APK sideloading for personal driver use (1–2 users). No Google Play publication required.
  - Driver Safety: Designed strictly for stationary use (parked, stopped at traffic lights, pickup/drop-off). No in-motion interaction.
  - Audio Testability: Speech pipeline must be testable without a physical microphone.
  - Zero-Trust Security: Zero secrets or customer data in the public Git repository.
- **Current Working Assumptions**:
  - Exact versions of Kotlin, AGP, Gradle, Compose Compiler, `compileSdk`, and `targetSdk` will be aligned together during Android scaffolding based on official compatibility tables.
  - **MVP currently has no backend requirement.** A backend is not prohibited, but will only be considered if concrete evidence later demonstrates a genuine need (e.g., security boundary, server-side processing, centralized telemetry).
  - Push-to-talk is the initial MVP candidate interaction model.
- **Open Decisions (To be resolved via Spikes)**:
  - `Spike-STT-01`: Built-in `SpeechRecognizer` vs. On-Device Whisper vs. Cloud STT under cabin noise.
  - `Spike-TRANS-01`: Google ML Kit On-Device Translation vs. Cloud LLM on driver domain idioms.
  - `Spike-TTS-01`: Android System TTS voice availability and loudness vs. Cloud TTS.
  - `Spike-AUDIO-01`: Push-to-Talk (Hold-to-speak) vs. Tap-to-Talk with Voice Activity Detection.
  - `Spike-HYBRID-01`: Feasibility, latency, and battery impact of an opportunistic offline-first hybrid fallback.

---

## Documentation Index

| Document | Purpose |
| :--- | :--- |
| [PROJECT_VISION.md](PROJECT_VISION.md) | Product identity, detailed user problems, UX layout, safety constraints, and phrasebook concepts. |
| [ENGINEERING_RULES.md](ENGINEERING_RULES.md) | Engineering principles, decision classification, verification matrix ("environment matches failure mode"), security rules, CI/CD, and Git discipline. |
| [AGENTS.md](AGENTS.md) | Operational guidelines, context management, and code verification gates for AI agents working in this repository. |
| [.agents/rules/safe-agent-workflow.md](.agents/rules/safe-agent-workflow.md) | Non-destructive Git and workspace safety rules. |
| [docs/logs/README.md](docs/logs/README.md) | Traceable engineering log for bugs, debugging investigations, failed spikes, and benchmark results. |

---

## Engineering Environment & Prerequisites

- **Development OS**: Ubuntu Linux (VMware virtualized development environment)
- **JDK Version**: Java 17 (Temurin recommended)
- **Android SDK**: Android API 29+ compatibility, command-line tools / Android Studio
- **Build System**: Gradle with Kotlin DSL (`build.gradle.kts`)

---

## Verification & Quality Philosophy

> *"The verification environment must match the failure mode."*

- **Logic bugs** are caught via fast JVM unit tests.
- **UI state regressions** are caught via Compose UI tests.
- **Hardware & permission issues** are caught on a physical Android device.
- **Noise robustness & speaker audibility** are validated through field evaluation in a real vehicle cabin.
- **A green CI pipeline is not proof that the product works in the real world.**
