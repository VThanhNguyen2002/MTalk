# ENGINEERING_RULES.md — Technical Foundation & Architectural Constraints

> **Repository**: Mtalk (Public Repository)  
> **Status**: Living Engineering Policy & Source-of-Truth  
> **Companion Rules**: [.agents/rules/safe-agent-workflow.md](file:///home/vietthanh/Projects/Projects/MTalk/.agents/rules/safe-agent-workflow.md)

---

## 1. Core Engineering Principles

1. **Evidence Before Architecture**: No library, framework version, pattern, or infrastructure component is adopted without documented justification, requirements analysis, or empirical benchmark data.
2. **Abstraction Follows Evidence**: Avoid anticipatory or speculative abstractions (e.g., dynamic plugin loaders, complex factories) until at least two concrete implementations or distinct test requirements justify them.
3. **Verification Environment Must Match Failure Mode**: Every feature, bug fix, or refactor must be verified at the appropriate layer of the verification pyramid. CI passes are not proof of real-world functionality.
4. **Context Integrity & Traceability**: Git and project documentation are the primary sources of truth. Ephemeral AI context must never replace persistent project records.
5. **Zero-Trust Security for Public Repositories**: Absolute prohibition against committing secrets, private credentials, or identifiable customer data.

---

## 2. Technical Decision Classification

To avoid mistaking working assumptions for permanent decisions, all technical decisions are categorized into five distinct classes:

### Category A: Established Project Requirements
*Non-negotiable product constraints and architectural baselines decided by project requirements:*
- **Platform Compatibility**: Minimum compatibility target is Android 10 (`minSdk = 29`).
- **UI Technology**: Android native UI implemented using Kotlin and Jetpack Compose.
- **Distribution Baseline**: Direct APK sideloading for 1–2 personal users. Google Play publication is not required.
- **Driver Safety Constraint**: Primary interaction is strictly stationary (parked, stopped at traffic lights, pickup/drop-off, delivery handoff). No in-motion phone manipulation.
- **Audio Testability Constraint**: The speech pipeline must be testable in CI without physical microphone hardware.
- **Public Repository Security**: Zero secrets (API keys, keystores, tokens, credentials) in Git; the client APK is not a secure secret container.
- **Privacy Standard**: Zero real passenger audio recordings, transcripts, or personal identifying information in the public repository.
- **Baseline CI**: Automated compile, lint, and unit test checks from Day 1 on GitHub Actions.

### Category B: Engineering Principles
*Methodological rules governing how software is designed, written, and verified:*
- **Meaning Preservation Over Keyword Matching**: Translation must preserve intent, conversational context, and tone rather than applying naive word-for-word substitution.
- **Verification Environment Matches Failure Mode**: Match logic tests to JVM, UI to Compose tests, permissions/lifecycle to devices, and noise/audibility to real vehicle environments.
- **Quality Gate Before Merge**: Mandatory inspection of source code, git diff review, automated checks, and documented trade-offs before accepting changes.
- **Traceable Incident Logging**: Meaningful bugs, debugging investigations, failed spikes, and benchmark results must be documented in `docs/logs/`.

### Category C: Current Working Assumptions
*Reasonable default directions subject to validation during scaffolding or spikes:*
- **Build Tooling Versions**: Exact versions of Kotlin, Android Gradle Plugin (AGP), Gradle, Compose Compiler, `compileSdk`, and `targetSdk` are working assumptions to be aligned together during Android scaffolding using official compatibility tables.
- **Audio Decoupling Pattern**: Using an interface-based abstraction for audio input (e.g., `AudioSource`) is an accepted design direction, but specific class names and hierarchy details will remain proportional to implementation evidence.
- **Interaction Model**: Push-to-talk (hold-to-speak) is the primary MVP candidate, to be verified against vehicle acoustic noise.
- **Dual Output**: Presenting high-contrast translated text alongside synthesized audio playback to mitigate cabin noise.
- **Driver Phrasebook**: Predefined deterministic phrases for high-frequency driver-passenger exchanges to eliminate latency and network dependencies.
- **Backend Architecture**: **MVP currently has no backend requirement.** The application directly leverages on-device capabilities and provider APIs where justified.

### Category D: Open Decisions & Planned Spikes
*Explicitly undecided architectural forks requiring empirical evidence from Spikes:*

1. **[OPEN DECISION 1] Speech-to-Text (STT) Provider**
   - *Candidates*: Android built-in `SpeechRecognizer`, on-device Whisper models, or cloud STT APIs.
   - *Evidence Needed*: Recognition accuracy for accented Vietnamese and conversational English under background traffic noise; latency; offline availability on Android 10 devices; binary footprint; RAM usage.
   - *Proposed Spike*: `Spike-STT-01` — Benchmark candidate STT engines against a synthetic audio suite simulating vehicle noise.
2. **[OPEN DECISION 2] Translation Engine**
   - *Candidates*: Google ML Kit On-Device Translation (`com.google.mlkit:translate`), Cloud Translation API, or Cloud LLM.
   - *Evidence Needed*: Contextual translation naturalness for driver-passenger idioms; cold and warm execution latency; offline behavior; resource overhead.
   - *Proposed Spike*: `Spike-TRANS-01` — Evaluate ML Kit against cloud translation on a standardized driver dialogue dataset.
3. **[OPEN DECISION 3] Text-to-Speech (TTS) Engine**
   - *Candidates*: Android system `TextToSpeech` (`android.speech.tts.TextToSpeech`) vs. Cloud TTS services.
   - *Evidence Needed*: Availability of Vietnamese voice data across device vendors; voice loudness, clarity, and intelligibility through phone loudspeakers in high-noise environments.
   - *Proposed Spike*: `Spike-TTS-01` — Measure TTS initialization latency, voice availability, and loudspeaker audibility.
4. **[OPEN DECISION 4] Audio Input Interaction Model**
   - *Candidates*: Push-to-talk (hold-to-speak) vs. tap-to-talk with automated Voice Activity Detection (VAD).
   - *Evidence Needed*: Rate of premature cutoffs caused by vehicle rumble, horn blasts, or road vibrations vs. ergonomic friction for driver and passenger.
   - *Proposed Spike*: `Spike-AUDIO-01` — Compare false cutoff and trigger rates under simulated vehicle noise.
5. **[OPEN DECISION 5] Offline vs. Hybrid Fallback Strategy**
   - *Candidates*: Pure offline pipeline vs. opportunistic cloud fallback on low confidence.
   - *Evidence Needed*: Reliability of confidence scores provided by candidate engines; latency penalty of falling back; cellular connection drop rate during trips.
   - *Proposed Spike*: `Spike-HYBRID-01` — Evaluate complexity, latency, and battery drain of confidence-based fallbacks.

### Category E: Future Options / Non-Goals
*Explicitly deferred or non-essential capabilities for the foundation and MVP:*
- **Dedicated Backend**: A backend (Go, Python, Firebase) is **not prohibited**, but will only be introduced if concrete evidence demonstrates a necessary requirement (such as a security boundary, centralized model serving, multi-client synchronization, user accounts, or centralized telemetry). No backend will be introduced merely for architectural appearance.
- **Broad Multi-Language Support**: Scoped strictly to Vietnamese ↔ English initially; expansion to additional languages is a future option.
- **Advanced OS Integrations**: Floating window overlays, background recording services, wake-word listeners, and accessibility service hooks are non-goals for MVP.
- **Commercial Infrastructure**: Billing, in-app purchases, and Google Play Store release automation are out of scope.
- **Release Signing Automation**: Automated GitHub release signing workflows will be deferred until the application build pipeline requires them.

---

## 3. Platform & Compatibility Baseline

- **`minSdk = 29` (Android 10)**: Non-negotiable floor ensuring support for user hardware while providing modern Scoped Storage and runtime permission models.
- **SDK & Build Versions**: Exact `compileSdk`, `targetSdk`, AGP, and Gradle versions will be selected as a coherent set during scaffolding based on official Android compatibility matrices.
- **Runtime Permissions**:
  - `android.permission.RECORD_AUDIO`: Requested strictly in the foreground when the user initiates speech features.
  - Background recording is prohibited; no background services or wake-lock recording loops will be created.

---

## 4. Audio Pipeline Testability

The application must be architected so that CI and automated unit testing do not require physical microphone hardware.

- An interface-based audio input abstraction (e.g., `AudioSource`) is an accepted design direction.
- Implementations representing hardware capture and deterministic test replay (e.g., WAV/PCM file replay) are conceptual examples, not frozen class names.
- Concrete classes, streaming mechanisms (such as Kotlin Coroutines `Flow`), and buffer formats will be designed proportionally to evidence during the audio scaffold.

---

## 5. Vehicle Environment & Driver Safety Policy

1. **Acoustic Testing vs. Driving Behavior**:
   - Audio robustness may be evaluated using realistic vehicle acoustic conditions, including pre-recorded audio from moving vehicles or simulated cabin noise.
   - **This must NEVER be interpreted as permission or product intent for the driver to manipulate or interact with the phone while driving.**
2. **Operational Safety Rule**:
   - Primary user interaction must occur only while the vehicle is stationary (parked, stopped at traffic lights, pickup/drop-off, delivery handoff).
   - The UI must feature large, clear controls to avoid requiring precision taps.

---

## 6. Testing & Verification Matrix

The engineering rule: **"Verification environment must match failure mode."**

| Failure Mode | Verification Environment | Required Verification |
| :--- | :--- | :--- |
| **Logic & State Transitions** | Local JVM / CI | Unit Tests (JUnit 4/5, MockK) |
| **Compose UI Rendering / Interaction** | JVM (Robolectric) / Instrumentation | Compose UI Tests (`createComposeRule`) |
| **Provider API Contracts & Parsing** | Local JVM / CI | Contract / Mocked Integration Tests |
| **Android Permissions & Lifecycle** | Emulator / Physical Device | Android Instrumentation Test |
| **Microphone Hardware & Audio Capture**| Real Android Device (API 29+) | Physical Device Manual / Instrumented Test |
| **Cabin Noise & Speech Recognition** | Recorded Cabin Audio / Field Testing| Field Evaluation & Benchmark Dataset |
| **TTS Speaker Audibility in Cabin** | Real-world Car / Bike Environment | Field Evaluation |
| **Network Loss / Instability** | Mock network interceptors / Airplane Mode | Error-path Integration Tests |
| **APK Sideload & Installation** | Physical Device (API 29+) | `adb install` / Package Installer Test |

---

## 7. CI/CD Standards

GitHub Actions will be utilized for automated verification from Day 1:
- **Triggers**: Push to `main` and all Pull Requests.
- **Steps**: Checkout code, set up JDK (Temurin 17), configure Gradle with caching, execute static analysis (`lint`), run JVM unit tests (`testDebugUnitTest`), and build debug APK artifact (`assembleDebug`).
- **Zero Secrets in CI**: Standard pull request checks must never depend on live third-party API credentials. Fakes and deterministic fixtures must be used.

---

## 8. Zero-Trust Security Policy (Public Repository)

Mtalk is a public repository. The following security rules are mandatory:

1. **Forbidden in Git**:
   - API keys, access tokens, and passwords.
   - Keystores (`*.jks`, `*.keystore`) and signing credentials.
   - Local configuration files containing credentials (`local.properties`, `.env`).
   - Customer or passenger audio recordings, transcripts, or PII.
2. **Local Secret Storage**:
   - Secrets belong strictly in `local.properties` (ignored by Git) or environment variables injected via build configuration.
3. **The APK is Not a Secret Vault**:
   - Sensitive server keys must never be embedded in client APKs, as they can be extracted via reverse engineering.
4. **Secret Leak Remediation Protocol**:
   - In case of accidental secret commit:
     1. Stop immediately.
     2. Revoke and rotate the compromised credential in the external provider console.
     3. Scrub the credential from Git history using `git-filter-repo` or BFG.
     4. Document the incident in `docs/logs/`.

---

## 9. Git Standards & Hygiene

- **Branching**: `main` is the primary trunk.
- **Inspections**: Always inspect `git status` before beginning work.
- **Commit Format**: Conventional commit syntax (`feat:`, `fix:`, `test:`, `docs:`, `chore:`, `spike:`).
- **Safety**: Never execute destructive commands that discard uncommitted work (`git reset --hard`, `git clean -fd`).

---

## 10. Primary Authoritative References

Every technical assertion in this document is backed by official engineering documentation:

- **Android SDK & API Changes**:
  - [Android 10 Behavior Changes](https://developer.android.com/about/versions/10/behavior-changes-all)
  - [Android Uses-SDK Element](https://developer.android.com/guide/topics/manifest/uses-sdk-element)
- **Permissions & Privacy**:
  - [RECORD_AUDIO Reference](https://developer.android.com/reference/android/Manifest.permission#RECORD_AUDIO)
  - [Android 10 Background Activity & Microphone Restrictions](https://developer.android.com/about/versions/10/privacy/changes)
- **Jetpack Compose & Testing**:
  - [Jetpack Compose Testing Guide](https://developer.android.com/develop/ui/compose/testing)
  - [Compose Compiler Gradle Plugin](https://developer.android.com/studio/build/kotlin-compiler-plugin)
- **Speech & Machine Learning**:
  - [Android SpeechRecognizer](https://developer.android.com/reference/android/speech/SpeechRecognizer)
  - [Android TextToSpeech](https://developer.android.com/reference/android/speech/tts/TextToSpeech)
  - [Google ML Kit Translation for Android](https://developers.google.com/ml-kit/language/translation/android)
- **APK Packaging & Signing**:
  - [Sign Your App with apksigner](https://developer.android.com/studio/publish/app-signing)
  - [apksigner Command-Line Reference](https://developer.android.com/tools/apksigner)
- **CI/CD Infrastructure**:
  - [GitHub Actions setup-java](https://github.com/actions/setup-java)
  - [Gradle Build Action for GitHub Actions](https://github.com/gradle/actions/tree/main/setup-gradle)
