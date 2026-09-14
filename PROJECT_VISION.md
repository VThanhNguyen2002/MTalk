# PROJECT_VISION.md — Mtalk: Translate for Drivers

## 1. Product Identity & Purpose

- **Product Name**: Mtalk
- **Subtitle**: Translate for Drivers
- **Target Platform**: Android (Minimum: Android 10 / API 29+)
- **Distribution Model**: APK Sideload (Direct installation)
- **Intended Users**: 1–2 personal users (Primary user: Taxi / Delivery driver)

### Real-World Problem
During passenger transport and delivery work, drivers regularly encounter foreign customers with whom they must communicate quickly, accurately, and without friction regarding:
- **Destinations & Navigation**: Pick-up points, drop-off spots, landmarks, specific building gates, route choices ("Đi thẳng", "Rẽ trái", "Rẽ phải").
- **Payment & Fare**: Total amount, cash vs. banking transfer/QR ("Tiền mặt hay chuyển khoản?"), lack of small change ("Tôi không có tiền lẻ"), tipping/change policy ("Bạn có thể giữ tiền thừa").
- **Status & Timing**: Arrival notice ("Tôi đến rồi"), waiting time ("Tôi đang đợi bạn", "Đợi tôi 5 phút"), traffic delays ("Tôi đang bị kẹt xe").
- **Delivery Instructions**: Doorstep drop-off, apartment security check-in, customer descent ("Bạn có thể xuống đây không?").
- **Short Conversational Exchanges**: Clarifications, confirmation of passenger identity, unexpected requests.

Mtalk is **not** built to compete commercially with general-purpose translation tools like Google Translate. It is engineered specifically around the constrained, high-noise, high-urgency context of driving and delivery operations.

---

## 2. Core Product Goals

1. **Solve a Real Daily Problem**: Enable seamless, unambiguous communication between a Vietnamese driver and non-Vietnamese passengers or delivery recipients.
2. **Practice Disciplined Software Engineering**:
   - Modern Android architecture with Kotlin and Jetpack Compose.
   - Decoupled audio processing and provider-agnostic interfaces.
   - Strict testing pyramid with CI/CD automation from Day 1.
   - Comprehensive documentation of failures, debug sessions, and benchmark trade-offs.
3. **Produce a Rigorous Portfolio / CV Project**:
   - Portfolio value is an organic consequence of solving a real problem with disciplined engineering practices, clean architecture, and thorough validation.
   - No "resume dressing" or vanity complexity.

---

## 3. Core Product Concept & Pipeline

The translation workflow follows a bidirectional conversational pipeline:

```
[ Driver (Vietnamese) ]                           [ Passenger (Foreign Language) ]
         │                                                        │
         ▼                                                        ▼
   [ Audio Input ]                                          [ Audio Input ]
         │                                                        │
         ▼                                                        ▼
[ Speech-to-Text (STT) ]                                  [ Speech-to-Text (STT) ]
         │                                                        │
         ▼                                                        ▼
   [ Translation ]                                          [ Translation ]
         │                                                        │
         ▼                                                        ▼
  [ Translated Text ]                                      [ Translated Text ]
         │                                                        │
         ▼                                                        ▼
[ Text-to-Speech (TTS) ]                                  [ Text-to-Speech (TTS) ]
```

### Key Principles:
- **Dual Output**: Always display translated text **and** play synthesized speech. In loud cabin environments where phone speakers are drowned out by traffic, the visual text acts as a reliable fallback.
- **Context-Aware Translation**: Translation must preserve intent, conversational context, and tone rather than applying naive word-for-word or keyword substitution (e.g., distinguishing "Can you pull over here for a second?" from fragmented keywords).
- **Text Input Fallback**: When audio recognition fails or background noise is overwhelming, manual text entry must be available without breaking the conversation flow.

---

## 4. UI / UX Direction

- **Stack**: Kotlin + Jetpack Compose (Declarative UI).
- **Driver-Passenger Split Screen**:
  - **Top Half (Passenger Facing)**: Inverted or passenger-oriented orientation with large, high-contrast text and a prominent, universally understandable speak button.
  - **Bottom Half (Driver Facing)**: Driver-oriented controls with quick-action phrasebook buttons and Vietnamese voice input.
- **Zero-Friction Passenger Onboarding**: A first-time passenger looking at the screen must understand how to interact within 5 seconds without reading a tutorial.
- **Interaction Model**:
  - **Push-to-Talk (Hold-to-Speak)** is the current working assumption for the MVP. Automatic silence detection / VAD is notoriously error-prone in vehicle cabins where engine idle, horn blasts, and road vibrations trigger false cutoffs.
  - Tap-to-start / tap-to-stop and VAD alternatives remain open decisions to be evaluated during field testing.

---

## 5. Operational Safety Constraints

- **Strictly Stationary Use**: The primary operating context is when the vehicle is **parked, stopped at traffic lights, or during pick-up/drop-off**.
- **No In-Motion Manipulation**: The app must never require or encourage the driver to read fine print or manipulate buttons while the vehicle is actively moving.
- **No Unsafe OS Workarounds**: Floating chat heads/overlays, background audio sniffing, and accessibility service hacks are explicitly out of scope for the MVP. They introduce significant battery, security, and Android lifecycle complications.

---

## 6. Real-World Environmental Challenges

| Environmental Factor | Impact on System | Engineering Mitigation |
| :--- | :--- | :--- |
| **Traffic Noise & Horns** | Corrupts audio input; premature VAD cutoffs | Push-to-talk interaction; noise-robust STT; text input fallback |
| **Speaker Limitations** | Passenger cannot hear TTS over engine/wind noise | High-contrast visual text display alongside audio playback |
| **Weak / Unstable Cellular Data** | Cloud API latency spikes or dropouts | On-device processing baseline (e.g., ML Kit); phrasebook cache |
| **Dialects & Accents** | Lower recognition accuracy for foreign names/slang | Conversational context hints; prompt engineering for LLM/cloud STT |
| **Passenger Hesitation** | Unfamiliarity with foreign apps | Extremely minimal UI, universal iconography, zero tutorial requirement |

> **Safety Notice on Acoustic Testing**: Audio robustness may be evaluated using realistic vehicle acoustic conditions, including recordings made in moving vehicles or simulated cabin noise. **This must never be interpreted as permission or product intent for the driver to interact with the phone while driving.**

---

## 7. The Phrasebook Concept

Repetitive operational phrases represent approximately 70–80% of routine driver-passenger exchanges. The phrasebook is an essential operational optimization:
- **Zero Latency**: Instantaneous translation lookup and cached TTS generation.
- **100% Deterministic**: No hallucinations, mistranslations, or network dependencies.
- **Driver Quick-Triggers**: One-tap access to critical messages:
  - "Tôi đến rồi" (I have arrived)
  - "Tôi đang đợi bạn ở sảnh" (I am waiting for you at the lobby)
  - "Đợi tôi 5 phút" (Please wait 5 minutes)
  - "Bạn có thể xuống đây không?" (Can you come down here?)
  - "Bao nhiêu tiền?" / "Tiền mặt hay chuyển khoản?" (Cash or transfer?)
  - "Tôi không có tiền lẻ" (I do not have small change)
  - "Bạn có thể giữ tiền thừa" (You can keep the change)
  - "Tôi đang bị kẹt xe" (I am caught in a traffic jam)

---

## 8. Provider-Agnostic Architecture Strategy

To prevent vendor lock-in and enable rigorous benchmarking:
- High-level domain contracts (`SpeechProvider`, `TranslationProvider`, `TtsProvider`, `AudioSource`) decouple core business logic from third-party SDKs.
- **"Abstraction follows evidence"**: Abstractions will be created strictly around concrete interfaces, avoiding over-engineered factories, service locators, or plugin buses until multi-provider requirements are validated.

---

## 9. Backend Strategy

- **MVP Baseline**: **MVP currently has no backend requirement.** Direct communication from the Android client to on-device libraries and external APIs (where justified) minimizes operational complexity and latency.
- **Future Considerations**: A dedicated backend (e.g., Go, Python, or Firebase) is **not prohibited**, but may only be introduced if concrete evidence demonstrates a necessary requirement (such as a security boundary for credentials, server-side processing, centralized model serving, multi-client synchronization, user accounts, or centralized telemetry). No backend will be introduced merely for architectural appearance.
