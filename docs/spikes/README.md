# MTalk MVP Discovery & Spike Planning

> **Project**: MTalk — Translate for Drivers  
> **Status**: Spike Planning Phase (Post-Scaffold Phase 1)  
> **Scope**: Discovery & Validation Roadmap — **No feature implementation or premature abstractions**  
> **Governing Principles**: [ENGINEERING_RULES.md](../../ENGINEERING_RULES.md), [PROJECT_VISION.md](../../PROJECT_VISION.md), [AGENTS.md](../../AGENTS.md)

---

## 1. MVP User Journey

The initial MVP targets the **smallest practical end-to-end interaction** between a Vietnamese taxi/delivery driver and a foreign passenger during a **safely stationary vehicle moment** (such as parked curbside, passenger pickup/drop-off, or waiting before departure).

```
[ Safely Stationary Driver ]                                 [ Foreign Passenger ]
        │                                                              │
        ├─ 1. Selects target language (Default: English)               │
        ├─ 2. Presses & holds Driver PTT button                        │
        ├─ 3. Speaks short utterance in Vietnamese                     │
        ├─ 4. Releases PTT button                                      │
        ├─ 5. Audio captured -> STT converts to Vietnamese text        │
        ├─ 6. Text translated to English                               │
        ├─ 7. UI displays high-contrast visual English card ──────────>│ 8. Reads screen
        └─ 8. Device speaker plays English TTS audio ─────────────────>│ 9. Hears speech
                                                                       │
                                                                       ├─ 10. Responds / Acts
                                                                       │      (Hands cash, shows QR,
                                                                       │       or taps Passenger PTT)
```

### Step-by-Step Flow:
1. **Language Selection**: App defaults to **Vietnamese <-> English**. Driver can toggle the target foreign language with a single tap if needed.
2. **Driver Utterance Capture**: Driver presses and holds the Push-to-Talk (PTT) button, speaks a short operational phrase (e.g., *"Bạn thanh toán tiền mặt hay chuyển khoản?"*), and releases the button.
3. **Speech-to-Text (STT)**: Audio stream converts into Vietnamese text.
4. **Translation**: Vietnamese text converts to English text preserving domain context.
5. **Dual Presentation**:
   - **Visual Output**: Translated English sentence renders immediately in massive, high-contrast typography facing passenger/driver.
   - **Auditory Output**: Phone speaker plays English Text-to-Speech (TTS) voice automatically.
6. **Passenger Response**:
   - *Physical action*: Passenger hands cash or displays banking app screen.
   - *Verbal exchange*: If verbal reply is needed, passenger taps/holds the inverted Passenger PTT button, speaks in English, and the driver receives Vietnamese text and audio.

### Journey Hypotheses & Operational Constraints:
- *Safety Constraint (Safely Stationary Only)*: Interaction occurs exclusively while the vehicle is safely stationary (parked curbside, passenger pickup/drop-off, waiting before departure). The application is explicitly not designed for interaction while the vehicle is operating or in motion.
- *Interaction Hypothesis (PTT vs. VAD)*: Manual press-and-hold (PTT) is hypothesized to be more reliable than automated silence detection (VAD) in vehicle cabins where cabin noise could trigger false cutoffs or unintentional recording.
- *Output Role Hypothesis (Visual & Audio)*: In-cabin noise or passenger earphones can make device speaker audio difficult to hear; high-contrast on-screen text serves as an essential visual communication medium alongside voice.
- *Initial Scope (Candidate Language Pair)*: Vietnamese <-> English is selected as the initial candidate language pair to evaluate pipeline feasibility.

---

## 2. Candidate MVP Capabilities

| Category | Capability Description | Rationalization & Boundary |
| :--- | :--- | :--- |
| **Must Validate**<br>*(Prior to implementation)* | **Cabin Audio Noise Robustness** | Determine whether STT accurately transcribes speech with engine idling, air conditioning, and street traffic noise. |
| | **PTT Interaction Ergonomics** | Evaluate whether touch-and-hold tactile feedback works reliably without clipping start/end syllables. |
| | **Domain Translation Accuracy** | Evaluate translation fidelity on short, idiomatic Vietnamese driver phrases (avoiding literal word-for-word misinterpretation). |
| | **Phone Speaker TTS Audibility** | Evaluate whether device TTS output is audible across cabin distances under typical background noise. |
| | **Latency Budget** | Measure provisional response times from PTT release to visual display and audio playback. |
| | **Network Degradation Behavior** | Verify error recovery and degradation when cellular connectivity is weak, high-latency, or absent. |
| **Candidate MVP**<br>*(Initial usable build)* | **Single-Screen Dual-Role UI** | Driver input area and passenger-facing high-contrast translated text card. |
| | **Manual Push-to-Talk (PTT)** | Explicit press-to-record, release-to-process interaction. |
| | **Vietnamese <-> English Pipeline** | Bi-directional voice-to-text-to-voice translation. |
| | **Pre-Defined Driver Operational Phrases** | Rapid one-tap triggers for common fixed operational phrases (e.g., Destination check, Cash/Transfer, Arrival notice, Waiting notice). |
| | **Explicit Visual State Transitions** | Unambiguous visual states: `IDLE`, `LISTENING`, `PROCESSING`, `RESULT`, `ERROR`. |
| | **Graceful Error UI** | On-screen text fallback and retry action when network, microphone, or speech recognition fails. |
| **Later**<br>*(Explicitly deferred)* | **Multi-Language Matrix** | Defer additional language pairs (Chinese, Korean, Japanese, European languages) until the English pipeline is benchmarked and verified. |
| | **Floating Bubble / System Overlay** | Defer system overlay (`SYSTEM_ALERT_WINDOW`) over driver navigation apps to avoid accessibility service complexity, aggressive OS termination, and driver distraction. |
| | **Continuous Background Listening** | Reject wake-word detection ("Hey MTalk") and continuous background audio capture for battery conservation and driver privacy. |
| | **Custom Dedicated Backend** | Defer backend server infrastructure until concrete requirements (such as server-side secret boundaries or custom heavy model hosting) are demonstrated. |
| | **User Accounts & Cloud Sync** | No user login, cloud profiles, or remote state synchronization for personal utility use. |
| | **Audio Recording Storage / History** | Reject permanent audio recording databases or conversation archives to protect passenger and driver conversational privacy. |

---

## 3. Spike Measurement & Ground-Truth Protocol

Before conducting comparative evaluations across speech, translation, and audio components, a standardized, privacy-safe measurement protocol is required to ensure reproducibility.

### Benchmark Inputs & Data Provenance
- **Driver-Authored / Synthetic Only**: Benchmark inputs must consist exclusively of driver-authored phrases, synthetic sentences, or explicitly consented non-passenger simulations.
- **Strict Privacy Boundary**: Real passenger audio recordings and passenger speech transcripts constitute private third-party personal data. Under zero-trust engineering rules, raw passenger audio recordings and unconsented live ride transcripts must **never** enter the public repository or serve as committed benchmark fixtures.
- **Reproducibility & Versioning**: Test sets must be maintained as versioned, deterministic text fixtures with clear provenance.

### Experiment Metadata Schema
Every benchmark execution record must capture:
- `dataset_id`: Identifier of the test fixture set.
- `dataset_version_or_hash`: Git hash or version identifier of the input fixture set.
- `language_pair`: Source and target languages (e.g., `vi-VN -> en-US`).
- `dialect_accent_strata`: Regional dialect/accent variation where relevant (Northern, Central, Southern Vietnamese).
- `device_model` & `android_api_level`: Test device specifications (e.g., Pixel 6, Android 14 / API 34).
- `audio_source`: Physical device microphone, direct digital WAV replay, or acoustic loudspeaker playback.
- `recording_conditions`: Environmental acoustic state (engine idle, AC on/off, window state, simulated exterior noise).
- `network_condition`: Network profile (WiFi, strong 4G, throttled 3G, offline).
- `provider_engine_model`: Candidate engine and version identifier where accessible.
- `test_date`: Timestamp of test run.
- `scoring_protocol`: Specific evaluation rubric applied.

### Component Evaluation Protocols
1. **STT Accuracy Protocol**:
   - Compare candidate STT output against the authoritative ground-truth transcript.
   - Record specific error categories: word substitutions, deletions, insertions, and domain-critical term corruptions (e.g., payment methods, numbers, destinations).
2. **Translation Evaluation Protocol**:
   - Evaluate translations against approved driver-authored operational phrases.
   - Apply a structured bilingual review procedure distinguishing semantic correctness (intent accurately preserved) from stylistic preference.
   - Separately categorize and track critical operational terms (fare, cash, transfer, destination, stop).
3. **Provider Comparison Control**:
   - Maintain identical, controlled input conditions across all candidate engines.
   - Stated test sample counts (such as 10, 15, 20, or 25 utterances) represent **provisional exploratory batch sizes** for initial spikes, not statistically established acceptance thresholds. Statistical significance is not claimed.

---

## 4. Unknowns & Assumptions Matrix

All numerical values listed below represent **provisional measurement targets**, **hypotheses**, or **candidate decision boundaries** to be evaluated empirically, rather than pre-established product requirements.

| ID | Assumption / Unknown | Why It Matters | Evidence Needed | Consequence If False |
| :--- | :--- | :--- | :--- | :--- |
| **U-01** | **Push-To-Talk (PTT) vs. Auto-VAD** | Vehicle cabins contain intermittent noise (horns, AC, music) that could cause false VAD cutoffs. | Audio capture comparison in vehicle acoustic conditions measuring syllable clipping vs. false VAD triggers. | If PTT touch-and-hold proves ergonomically awkward, evaluate two-tap (tap-start, tap-stop) interaction. |
| **U-02** | **STT Accuracy in Vehicle Noise** | High transcription error rates make subsequent translation inaccurate or misleading. | Benchmark transcription accuracy on driver-authored test utterances under simulated and real cabin acoustic noise. | If lightweight on-device STT fails under noise, evaluate cloud STT or noise-suppression preprocessing. |
| **U-03** | **Translation Quality on Driver Idioms** | Driver operational phrases frequently use colloquial abbreviations (*"chuyển khoản"*, *"tiền lẻ"*, *"kẹt xe"*). | Evaluate semantic translation fidelity of driver-authored operational sentences across candidate translation engines. | If translation engine fails on domain idioms, introduce a deterministic local template/phrasebook layer. |
| **U-04** | **TTS Audibility & Intelligibility** | If the passenger cannot clearly hear speech output from the rear seat, voice output provides little utility. | Audibility and clarity observation from driver dashboard mount to rear passenger seat with windows up and down. | If device speaker audio is inaudible, the high-contrast visual text card becomes the primary communication medium. |
| **U-05** | **End-to-End Latency Target** | Lengthy interaction delays hinder natural curbside communication. Provisional target: `< 2.0s` under favorable network conditions. | Cumulative timestamping across pipeline stages: audio capture release -> STT result -> translation -> TTS preparation -> UI render. | If latency exceeds acceptable conversational pacing, present immediate intermediate visual state and reduce network roundtrips. |
| **U-06** | **Network Dependency & Degradation** | Vehicles operate in areas with variable connectivity (underground parking, tunnels, cellular dead zones). | Error-path testing simulating high packet loss, throttled cellular data, and offline state. | Application must provide immediate visual offline feedback and fall back to local operational phrases without hanging. |
| **U-07** | **Language-Pair Scope (VI <-> EN)** | Supporting numerous languages early risks diluting translation verification quality. | Evaluation of whether Vietnamese <-> English sufficiently covers core foreign passenger interactions for MVP validation. | If English proves insufficient for primary foreign passenger scenarios, evaluate Korean and Chinese next. |
| **U-08** | **Microphone Permission Handling** | Permission denial or revocation must not lead to silent application failure or crash. | Android permission denial and permanent denial ("Don't ask again") flow verification. | Application must present a clear explanation of microphone necessity with direct navigation to system app settings. |
| **U-09** | **UI Readability Under Daylight Glare** | High ambient light and dashboard viewing distance make small text unreadable. Provisional target: high contrast and large text hierarchy. | Visual legibility inspection on a physical device mounted on a vehicle dashboard under direct sunlight. | Adjust text sizing, weight, and background contrast to ensure legibility from standard passenger viewing angles. |
| **U-10** | **Failure & Retry Behavior** | Transient network or recognition errors should not clear driver input or freeze the UI. | Failure recovery testing under forced API errors, timeout events, and empty speech captures. | Application must preserve the original transcript and provide an immediate, single-tap retry action. |
| **U-11** | **Operational Phrasebook Utility** | Repetitive operational phrases may satisfy common stationary coordination scenarios with low latency. | Frequency review of common driver coordination scenarios using driver-authored phrase sets. | If free-form speech recognition is inconsistent or slow under noise, operational phrasebook becomes the core MVP mechanism. |
| **U-12** | **Need for Dedicated Backend** | Introducing a backend server increases operational cost, latency, and maintenance overhead. Current working assumption: evaluate client-only first. | Architectural assessment of whether on-device engines or direct provider APIs satisfy MVP constraints without a proxy. | If direct integration cannot protect necessary credentials or achieve required processing, backend architecture is evaluated. |

---

## 5. Spike Plan & Sequencing

Spikes must proceed according to the dependency structure below. The **Measurement & Ground-Truth Protocol** is a prerequisite for all quantitative evaluations. Independent text-only or audio-only experiments may proceed in parallel once ground-truth fixtures are defined.

```
       [ Measurement & Ground-Truth Protocol ]
                          │
          ┌───────────────┴───────────────┐
          ▼                               ▼
 [ Spike-AUDIO-01: PTT ]       [ Spike-TRANS-01: Translation ]
          │                               │
          ▼                               │
 [ Spike-STT-01: Speech-to-Text ]         ▼
          │                    [ Spike-TTS-01: Text-to-Speech ]
          │                               │
          └───────────────┬───────────────┘
                          ▼
        [ Spike-LATENCY-01: End-to-End & Network ]
```

*Note on parallelizability*:
- `AUDIO-01` and `STT-01` share the same acoustic recording protocol.
- `TRANS-01` evaluates text translation independently using approved text fixtures.
- `TTS-01` evaluates speech synthesis independently using approved translated text fixtures.
- `LATENCY-01` evaluates the candidate end-to-end pipeline once candidate components are selected.

---

### Spike-AUDIO-01: Push-To-Talk (PTT) Ergonomics & Audio Capture
- **Question**: Does manual PTT (press-and-hold) capture complete utterances without clipping opening syllables or trailing words, compared to tap-to-start / tap-to-stop?
- **Hypothesis**: Press-and-hold with a provisional pre-capture buffer avoids initial syllable clipping and avoids capturing trailing ambient noise.
- **Minimal Experiment**: Isolated Android test harness capturing audio via `AudioRecord` / MediaRecorder using (a) Touch Down -> Touch Up vs. (b) Tap Start -> Tap Stop. Test with a provisional batch of 10 driver-authored phrases spoken immediately upon touch.
- **Evidence to Collect**: Audio start/end clipping rate, perceived interaction ease, accidental cancellation rate.
- **Candidate Decision Boundary**: If Touch Down clips initial speech in > 5% of test utterances in the exploratory batch, evaluate tap-to-talk with visual confirmation.
- **Architectural Consequence**: Informs audio input state transitions in the UI layer and audio capture buffer contracts.

### Spike-STT-01: Speech-to-Text Engine Evaluation
- **Question**: What transcription accuracy and latency can candidate STT options achieve for Vietnamese driver phrases under cabin acoustic conditions?
- **Hypothesis**: Android built-in `SpeechRecognizer` platform API provides low latency without bundling models, but alternative on-device or cloud models may exhibit different noise resilience that must be tested.
- **Platform Clarification**: Android `SpeechRecognizer` is a platform API whose underlying service and behavior depend on device configuration (it may use on-device recognition or network recognition depending on OEM and Google Play services availability).
- **Minimal Experiment**: Benchmark transcription using a provisional exploratory set of 15 driver-authored utterances played against simulated vehicle acoustic noise (engine idle, AC, passing traffic):
  1. Android platform `SpeechRecognizer`.
  2. Candidate local on-device model (if packaging and runtime allow).
  3. Candidate cloud STT API.
- **Evidence to Collect**: Transcription accuracy compared against ground truth, execution latency (release to final text), package size impact, offline availability.
- **Candidate Decision Boundary**: Evaluate platform `SpeechRecognizer` first. If transcription accuracy on core operational phrases is acceptable under vehicle noise, adopt it to avoid model packaging overhead. If error rates prevent reliable translation, evaluate cloud or bundled alternatives.
- **Architectural Consequence**: Determines whether a `SpeechProvider` abstraction is justified or if the platform API is directly usable.

### Spike-TRANS-01: Translation Engine Evaluation
- **Question**: Can on-device translation (such as Google ML Kit Translate) handle short driver operational idioms with acceptable semantic fidelity, or is cloud translation required?
- **Hypothesis**: On-device ML Kit avoids network transit latency, but its local model inference latency and translation quality on colloquial Vietnamese ride-hailing terms must be compared against cloud translation alternatives.
- **Performance & Cost Clarification**: On-device translation eliminates network roundtrips but still incurs local inference time, model download footprint, and device memory usage; it must not be assumed to have "0ms latency" or "zero cost".
- **Minimal Experiment**: Translate a provisional fixture of 25 driver-authored operational phrases (e.g., *"Tiền mặt hay chuyển khoản?", "Tôi đợi ở sảnh B nhé", "Bị kẹt xe tầm 10 phút"*) through:
  1. Google ML Kit On-Device Translation (Vietnamese -> English).
  2. Candidate cloud translation service / LLM API.
- **Evidence to Collect**: Semantic translation correctness (scored via bilingual review protocol), local inference latency (ms), network roundtrip latency (for cloud), model storage footprint.
- **Candidate Decision Boundary**: If on-device ML Kit achieves acceptable semantic accuracy on core fare and destination phrases, prefer it for offline capability. If critical operational terms are consistently distorted, evaluate cloud translation with local phrasebook fallback.
- **Architectural Consequence**: Determines whether offline-first translation architecture is viable or if network connectivity is required.

### Spike-TTS-01: Text-to-Speech Engine & Cabin Audibility
- **Question**: Is native Android `TextToSpeech` output intelligible from the dashboard mount to the rear passenger seat under typical vehicle interior noise?
- **Hypothesis**: Android platform `TextToSpeech` output volume and voice quality require experimental verification in a vehicle interior; maximum volume setting alone does not guarantee passenger comprehension over road noise.
- **Minimal Experiment**: Synthesize 5 translated English operational phrases using native Android `TextToSpeech` on a physical device mounted on a vehicle dashboard. Observe intelligibility from the rear passenger seat with windows up and down.
- **Evidence to Collect**: Intelligibility observation from rear seat, speech synthesis startup latency, language voice availability across test devices.
- **Candidate Decision Boundary**: If platform TTS is clearly intelligible from the rear seat, adopt native `TextToSpeech` directly. If intelligibility is poor, treat the visual text card as the primary communication medium.
- **Architectural Consequence**: Determines whether custom audio synthesis libraries are needed or if platform `TextToSpeech` suffices.

### Spike-LATENCY-01: End-to-End Pipeline Latency & Network Degradation
- **Question**: What is the cumulative end-to-end turnaround time from PTT release to display/playback, and how does the pipeline degrade on poor cellular connections?
- **Hypothesis**: A candidate pipeline combining speech, translation, and audio output can complete within a provisional `< 2.0s` target under favorable connectivity, but network-dependent operations degrade significantly under weak signal.
- **Minimal Experiment**: Measure cumulative timing on real hardware over WiFi, strong cellular (4G), and simulated throttled/lossy connectivity (relevant only if network-dependent services are selected).
- **Evidence to Collect**: Cumulative timeline: audio buffer flush -> STT -> translation -> TTS preparation -> UI render. Failure and timeout rates.
- **Candidate Decision Boundary**: Total pipeline turnaround should target `<= 2.0s` under normal connectivity. UI must provide immediate visual transition within 100ms of PTT release.
- **Architectural Consequence**: Dictates UI loading states, threading dispatch, and timeout cancellation policies.

---

## 6. User Real-Device Validation

The developer will personally validate candidate builds on a **physical Android device (API 29+)**.

> [!IMPORTANT]
> **Validation Context**:
> - Testing must occur exclusively during **safely stationary vehicle moments** (parked curbside, passenger pickup/drop-off, waiting before departure).
> - All numerical values below are **provisional measurement targets** to guide testing methodology, not pre-established product requirements.
> - Testing on a single physical device does not establish broad Android API-level behavior across hardware variations; device model and OS version must be recorded.
> - Private real-world observations must remain strictly separate from repository-safe public benchmark fixtures.

### UI/UX
- **Readability**: Translated text card must be legible from standard passenger and driver viewing angles (provisional distance: ~1 meter) under direct sunlight and evening ambient lighting.
- **Touch Target Ergonomics**: PTT button touch area should be generous (provisional target: `>= 72dp x 72dp`) for reliable activation without divided visual attention.
- **Visual Feedback**: Screen must unambiguously reflect operational states:
  - `IDLE`: Clear call-to-action indicating active language pair and ready state.
  - `RECORDING`: High-visibility border or visual indicator confirming active audio capture.
  - `PROCESSING`: Distinct progress indicator without UI thread freezing.
  - `RESULT`: Large bilingual text presentation with manual audio replay trigger.
  - `ERROR`: Clear error message with single-tap retry action.

### Performance (Provisional Targets)
- **Cold App Startup**: Target `< 1200ms` from launch to interactive `IDLE` state.
- **Audio Capture Initialization**: Target `< 50ms` from button touch to active audio capture.
- **Perceived Responsiveness**: Immediate visual feedback upon button press and release; zero frame drops on UI thread.
- **End-to-End Turnaround**: Target `< 2000ms` from PTT release to translated display and audio playback under normal network conditions.

### Resource Behavior
- **CPU Utilization**: CPU activity should remain confined to active processing; idle CPU target `< 2%`.
- **Memory Footprint**: Stable memory allocation; verify absence of resource leaks over an exploratory test run of 50 repeated translation cycles.
- **Battery & Thermal Behavior**: Observe device temperature and battery drain during a 30-minute operational session.

### Real-World Audio Scenarios
1. *Vehicle stationary with engine idling and air conditioning operating*.
2. *Curbside pickup with exterior motorcycle and traffic noise*.
3. *Rain falling on the vehicle windshield*.
4. *Driver voice projection variations (conversational vs. low volume)*.
5. *Vietnamese regional dialect variations (Northern, Central, Southern accents)*.

### Network Conditions
- **Normal Connectivity (WiFi / 4G)**: Baseline reference for optimal turnaround.
- **Throttled / Congested Cellular**: Verify behavior when requests experience latency; provisional timeout boundary: 5 seconds before user-facing error.
- **Network Transition**: Verify handling when transitioning between WiFi and cellular during operation.
- **Offline / Airplane Mode**: Immediate visual notification ("No network connection") without hanging on socket calls (applicable if network services are used).

### Failure Modes & Edge Cases
- **Microphone Permission Denied**: Clear explanation of necessity; direct navigation to system app settings if permanently denied.
- **Microphone Hardware Contention**: Graceful error handling if the microphone is occupied by an ongoing phone call.
- **Brief / Accidental Tap**: Tapping PTT briefly (provisional threshold: `< 300ms`) or capturing silence displays *"No speech detected"* without failing.
- **Low Confidence / Unclear Audio**: Prompts user with *"Could not understand, please try again"* rather than displaying garbled text.

### Driver Workflow & Interference
- **Navigation App Coexistence**: Task-switching between driver navigation apps (Grab Driver, Google Maps) and MTalk must preserve application state.
- **Stationary Safety Adherence**: Workflow must be completely executable during brief stationary pauses without prolonged attention.

---

## 7. Evaluation & Evidence Format

All spike experiments and real-device test runs must record results in an isolated log file under `docs/logs/` following this standardized schema:

```markdown
# Experiment Log: [Spike-ID] - [Test Description]

- **Date**: YYYY-MM-DD
- **Dataset ID & Version**: [e.g., fixture-driver-phrases-v1 (git hash)]
- **Device**: [e.g., Pixel 6 / Samsung Galaxy S21 / Xiaomi Note 10]
- **OS Version**: Android [e.g., 12 (API 31) / 14 (API 34)]
- **Scenario**: [Cabin Idle / Curbside Noise / Stationary Rain]
- **Language Pair**: Vietnamese -> English
- **Audio Source**: [Physical Device Mic / Direct Digital WAV Replay]
- **Engine / Model**: [Candidate Engine and Version]
- **Network Condition**: [WiFi / 4G / Throttled 3G / Offline]

### Test Utterances & Results

| # | Spoken Input (Ground Truth) | Candidate STT Output | Candidate Translation Output | Latency (ms) | Status | Notes / Failure Cause |
|---|-----------------------------|----------------------|------------------------------|--------------|--------|-----------------------|
| 1 | "Bạn thanh toán tiền mặt hay chuyển khoản?" | ... | ... | ... | PASS/FAIL | ... |
| 2 | "Đợi tôi khoảng năm phút nhé" | ... | ... | ... | PASS/FAIL | ... |

### Observations & Failure Analysis
- Audio artifacts observed:
- Misrecognized words or error categories:
- Acoustic interference details:

### Conclusion & Architectural Verdict
- [ ] Hypothesis Supported
- [ ] Hypothesis Refuted
- **Decision**: [Actionable decision following decision gates]
```

> [!CAUTION]
> **Data Privacy Rule**:
> - Real passenger audio recordings and live ride transcripts are private third-party data.
> - Raw passenger audio or unconsented transcripts must **never** be committed to the public Git repository.
> - All committed benchmark fixtures must have verified provenance (synthetic, driver-authored, or explicitly consented simulations).
> - Provider outputs generated from approved synthetic/driver-authored fixtures may be recorded in evaluation logs.

---

## 8. Architecture Decision Gates

To enforce **"Abstraction follows evidence"** and avoid premature complexity, the following evidence gates govern component introduction:

1. **Gate 1 — Speech Engine Abstraction**:
   - Do NOT introduce a `SpeechProvider` abstraction or factory until spike evidence demonstrates that provider substitution, test isolation, or an external engine boundary is concretely required.
2. **Gate 2 — Translation Provider Abstraction**:
   - Do NOT introduce a `TranslationProvider` abstraction until spike evidence demonstrates that multi-provider fallback, runtime engine substitution, or test isolation is concretely required.
3. **Gate 3 — Audio Output Abstraction**:
   - Do NOT introduce custom audio playback abstractions until spike evidence demonstrates that Android platform `TextToSpeech` fails audibility, format support, or functional requirements.
4. **Gate 4 — Backend Infrastructure**:
   - Do NOT introduce backend servers, cloud functions, or API gateways until a concrete requirement (such as third-party secret protection that cannot reside on-device, custom heavy model hosting, or shared state) is proven unavoidable.
5. **Gate 5 — Persistent Storage**:
   - Do NOT introduce a persistent local database (Room/SQLite) until a concrete persistence requirement (such as user-customized phrasebooks or persistent operational logs) is demonstrated.
6. **Gate 6 — Dependency Injection Framework**:
   - Do NOT introduce a Dependency Injection framework (Hilt, Koin, Dagger) until dependency graph complexity creates a demonstrated maintenance or testability bottleneck.

---

## 9. Out of Scope for Current Checkpoint

This discovery checkpoint is an architectural and planning deliverable. The following components are **explicitly out of scope**:
- No implementation of STT, TTS, or Translation logic.
- No network requests, HTTP clients, or API integrations.
- No backend code or cloud infrastructure.
- No database, local storage, or file caching.
- No Dependency Injection frameworks (Hilt/Koin).
- No Jetpack Navigation library or multi-screen routing.
- No changes to Gradle build scripts, versions, or dependencies.
- No load-testing frameworks (k6, JMeter); load testing is deferred until a multi-user backend actually exists.

---

## 10. Decision Records (Established Baseline)

### ADR-001: Mobile Target Platform & Distribution Model
- **Context**: Mobile platform selection and delivery mechanism for MTalk driver utility.
- **Options**: (A) Android native app with APK sideloading; (B) Cross-platform (Flutter/React Native); (C) Progressive Web App (PWA).
- **Project Decision**: Android 10+ (API 29+), Kotlin, Jetpack Compose, distributed via direct APK sideloading for personal driver use.
- **Rationale**: Android native provides direct integration with platform audio hardware and TTS services without intermediate cross-platform bridge layers. Direct APK distribution avoids commercial store publishing overhead for personal operational use.
- **Trade-offs**: Manual APK updates; no iOS support.
- **Verification**: Built and packaged `app-debug.apk` targeting API 36 with `minSdk 29` in Scaffold Phase 1.
- **Revisit Condition**: If non-Android driver hardware is adopted or commercial store distribution is requested.

### ADR-002: UI Framework
- **Context**: UI framework selection for reactive, minimal-boilerplate interface.
- **Options**: (A) Jetpack Compose; (B) Traditional Android XML Views with ViewBinding.
- **Project Decision**: Jetpack Compose with Kotlin 2.4.20 and Compose BOM 2026.04.01.
- **Rationale**: Jetpack Compose is the modern official Android toolkit, offering concise declarative UI state modeling for dynamic bilingual cards without XML layout overhead.
- **Trade-offs**: Requires modern Kotlin compiler toolchain.
- **Verification**: Compose UI rendered successfully in `MainActivity` during Scaffold Phase 1.
- **Revisit Condition**: None.

### ADR-003: Single-Module Architecture for MVP
- **Context**: Project modularization strategy for initial scaffold and exploratory spikes.
- **Options**: (A) Single `:app` module; (B) Multi-module by feature (`:feature:stt`, `:feature:translation`, `:core:audio`).
- **Project Decision**: Single `:app` module for Scaffold Phase 1 and initial exploratory spikes.
- **Rationale**: Multi-module builds introduce configuration complexity, slower Gradle turnaround, and rigid boundary maintenance before domain contracts are validated by empirical evidence.
- **Trade-offs**: Monolithic package layout; requires disciplined package boundaries to facilitate future modularization if needed.
- **Verification**: Verified clean build and fast Gradle turnaround (`./gradlew check` in ~3s).
- **Revisit Condition**: Revisit if build durations increase significantly or independent component boundaries are empirically justified.

### ADR-004: Backend Architecture (Current Working Assumption: Deferred)
- **Context**: Architectural placement of translation and speech processing pipelines.
- **Options**: (A) Custom backend microservice / proxy; (B) Direct client-to-provider or on-device processing.
- **Current Working Assumption**: Defer backend creation. Evaluate on-device engines and direct provider integration first.
- **Rationale**: A dedicated backend introduces server hosting costs, maintenance overhead, and additional network latency hops. Client-side evaluation should precede server introduction under YAGNI principles.
- **Open Architectural Questions**: API key protection for commercial cloud services (if cloud APIs are chosen) and whether third-party credential boundaries necessitate a lightweight proxy.
- **Trade-offs**: If cloud APIs requiring confidential keys are adopted, a proxy boundary may become necessary.
- **Verification**: Scaffold Phase 1 operates completely standalone without backend dependencies.
- **Revisit Condition**: Revisit after `Spike-STT-01` and `Spike-TRANS-01` if selected engines cannot operate securely or reliably directly from the client.
