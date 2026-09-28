# MTalk Measurement & Ground-Truth Protocol

> **Project**: MTalk — Translate for Drivers<br>
> **Status**: Operational Specification (Spike Phase 1 Prerequisite)<br>
> **Governing Principles**: [ENGINEERING_RULES.md](../../ENGINEERING_RULES.md), [PROJECT_VISION.md](../../PROJECT_VISION.md), [AGENTS.md](../../AGENTS.md), [docs/spikes/README.md](README.md)<br>
> **Purpose**: Standardized, privacy-safe evidence generation and evaluation protocol for exploratory spikes.

---

## 1. Operating Principle & Scope

This protocol defines how future MTalk spikes (AUDIO-01, STT-01, TRANS-01, TTS-01, LATENCY-01) create, record, and evaluate empirical evidence.

**Core Principles**:
- **Evidence before architecture**: Architectural decisions and abstractions must be justified by documented empirical observations, not theoretical assumptions.
- **Specification, not implementation**: This document defines the measurement and recording standard. It does not implement test harnesses, benchmark runners, or data pipelines.
- **Zero-trust data privacy**: Live passenger audio and unconsented transcripts are strictly prohibited across all repositories and artifacts.

---

## 2. Allowed Data Sources & Privacy Boundary

To maintain zero-trust security and protect third-party privacy, test inputs are strictly restricted by provenance.

### Permitted Data Sources
1. **Driver-Authored Content**: Phrases authored directly by drivers or team members representing authentic operational scenarios.
2. **Synthetic / Generated Content**: Structured sentences generated for phonetic coverage, boundary testing, or vocabulary variation.
3. **Consented Non-Passenger Simulations**: Controlled test audio spoken by team members or consenting volunteers simulating driver/passenger roles.

### Explicitly Prohibited Data Sources
The following data categories must **never** be collected, stored, processed, or committed:
- **Passenger Audio Recordings**: No audio captured from commercial passengers.
- **Live Ride Transcripts**: No transcripts or notes taken from active commercial rides without formal third-party consent.
- **Personally Identifiable Information (PII)**: No real passenger names, phone numbers, addresses, account IDs, payment cards, or sensitive personal data.
- **Uncontrolled Third-Party Recordings**: No eavesdropped conversations or ambient public recordings without consent.

> [!CAUTION]
> **Repository Privacy Rule**:
> Real passenger data is strictly barred from all Git history, fixtures, evaluation logs, screenshots, and test artifacts. All committed fixtures must have verifiable, approved provenance.

---

## 3. Fixture Identity Schema

Every test fixture must possess a deterministic, stable identifier to enable reproducibility across test runs.

| Field | Type | Description | Example |
| :--- | :--- | :--- | :--- |
| `fixture_id` | String | Unique permanent identifier (`MTALK-[SRC]-[TGT]-[SEQ]`) | `MTALK-VI-EN-001` |
| `fixture_version` | String | Semantic version of fixture text/definition | `1.0.0` |
| `source_hash_sha256` | String | SHA-256 hash of `source_utterance` UTF-8 bytes | `d1fe9f...` |
| `source_type` | Enum | `driver_authored`, `synthetic`, `consented_simulation` | `driver_authored` |
| `source_language` | String | BCP-47 source language tag | `vi-VN` |
| `target_language` | String | BCP-47 target language tag | `en-US` |
| `dialect_accent` | String | **Audio stage only.** Regional dialect/accent of the actual recorded speaker. Not required for text-only fixtures; must be declared when an audio recording is created against this fixture. | `vi-Southern` |
| `scenario` | String | Target operational context | `Fare payment method` |
| `intended_meaning` | String | Core operational intent / ground-truth semantics | `Driver asks cash or transfer` |
| `criticality` | Enum | `critical`, `operational`, `normal` (see Section 5) | `critical` |

---

## 4. Ground Truth Definitions

Evaluation requires distinguishing between surface wording and semantic operational intent. Ground truth is defined across separate levels:

```
[ Spoken Input ] ──> [ Expected Transcription ]  (Orthographic text)
                            │
                            ├──> [ Intended Meaning ]         (Semantic intent)
                            │
                            ├──> [ Expected Translation ]      (Target language text)
                            │
                            ├──> [ Operational Action ]       (Real-world consequence)
                            │
                            └──> [ Expected TTS Text ]        (Synthesizer input string)
```

1. **Source Utterance**: The exact text prompt provided to the speaker, or the authoritative written form of the utterance.
2. **Expected Transcription**: The authoritative orthographic text representation of the source utterance, including standard punctuation and accent marks. *For text-only fixtures where no audio recording exists, `source_utterance` serves as the provisional expected transcription. `expected_transcription` becomes an explicit, separate field in the fixture record when an audio recording is created and the recorded speaker's natural delivery may differ from the written source text. Any such difference requires a fixture version increment.*
3. **Intended Meaning**: The underlying operational intent independent of phrasing (e.g., "confirm drop-off location").
4. **Expected Translation**: Authoritative target-language text translation preserving operational intent.
5. **Expected Operational Action**: The required real-world physical or verbal response from the receiver (e.g., passenger shows QR code, driver confirms stop).
6. **Expected TTS Text**: The exact textual input provided to the speech synthesis engine.

*Distinction Rule*: **Exact wording match and semantic equivalence are distinct criteria.** A translation differing in word choice may still satisfy operational equivalence (e.g., *"Cash or transfer?"* vs. *"Do you pay by cash or bank transfer?"*). Both must be recorded.

---

## 5. Critical Phrases Protocol

Not all utterances have equal failure costs. Errors in pleasantries are inconvenient; errors in payment amounts or safety instructions can create disputes or safety hazards.

### Classification:
- **`critical`**: Utterances where misunderstanding directly impacts payment, destination, safety, or legal compliance (e.g., *"50 nghìn đồng"*, *"Rẽ phải ở ngã tư"*, *"Bạn thanh toán tiền mặt hay chuyển khoản?"*). Individual pass/fail tracking required for every critical-term within the utterance.
- **`operational`**: Utterances coordinating normal ride flow where errors disrupt coordination but do not constitute safety, payment, or destination failures (e.g., *"Tôi đợi ở sảnh B"*, *"Bị kẹt xe khoảng 5 phút"*).
- **`normal`**: Conversational, courtesy, or non-operational phrases (e.g., *"Chào bạn"*, *"Cảm ơn bạn"*). Errors are inconvenient but do not affect the trip outcome.

### Evaluation Rule for Critical Phrases:
- **Aggregate metrics are insufficient**: An aggregate Word Error Rate (WER) of 5% or BLEU score of 40 does not establish system readiness if critical operational phrases fail.
- Critical phrases must be evaluated with **individual pass/fail tracking**. Any distortion of a critical term (e.g., numeric amount, payment method, turning direction) constitutes a `Critical` error regardless of overall sentence similarity.

---

## 6. Execution Metadata Schema

### Audio Metadata (For Acoustic Experiments)
When evaluating audio capture, microphone hardware, or speech recognition, record:
- `device_model`: OEM and hardware model (e.g., `Google Pixel 6`, `Samsung Galaxy A52`).
- `android_version`: Android OS version and API level (e.g., `Android 14 (API 34)`).
- `audio_source`: Capture hardware (e.g., `device_built_in_bottom_mic`, `wired_headset_mic`, `direct_digital_wav`).
- `recording_environment`: Physical space (e.g., `stationary_sedan_cabin`, `stationary_motorbike_curbside`, `quiet_office`).
- `acoustic_condition`: Noise state (e.g., `engine_idling_ac_medium`, `heavy_exterior_traffic`, `rain_on_windshield`).
- `speaker_position`: Spatial relationship (e.g., `dashboard_mount_to_driver_50cm`, `dashboard_to_rear_seat_120cm`).
- `background_noise_description`: Qualitative noise character (e.g., continuous low-frequency rumble, intermittent motorcycle horns).
- `recording_format`: Audio parameters where relevant (e.g., `16kHz_16bit_mono_PCM_WAV`).

### Provider Metadata (For Engine Experiments)
When evaluating speech, translation, or synthesis providers, record:
- `provider_name`: Vendor or engine origin (e.g., `Android_Platform`, `Google_ML_Kit`, `OpenAI`, `Cloud_API`).
- `product_api`: Specific API or library (e.g., `android.speech.SpeechRecognizer`, `com.google.mlkit:translate:17.0.3`).
- `model_version`: Exposed model name or timestamp (e.g., `whisper-1`, `embedded_default`, `not_exposed`).
- `configuration`: Execution settings (e.g., `offline_only=true`, `temperature=0.0`, `sampling_rate=16000`).
- `test_timestamp`: ISO 8601 execution time.
- `network_condition`: Connection profile (e.g., `WiFi_stable`, `Cellular_4G_strong`, `Throttled_3G_lossy`, `Offline`).
- `client_commit`: Git commit hash of test harness or application build.
- `sdk_version`: Version of integrated client library if applicable.

*Note*: Providers vary in transparency; fields not exposed by a given platform API must be recorded as `not_exposed` rather than assumed.

---

## 7. Component Evaluation Categories

Evaluation logs must record structured, categorized observations rather than open-ended commentary. Categories are extensible as new failure modes emerge.

### STT Evaluation Categories
- `substitution`: Recognized word replaced ground-truth word (e.g., *"tiền"* -> *"tiệm"*).
- `insertion`: Extra word inserted that was not spoken.
- `deletion`: Spoken word omitted from transcript.
- `critical_term_error`: Any substitution, insertion, or deletion affecting a fare number, payment method, or destination keyword.
- `hallucination`: Output generated during silence or ambient background noise.
- `clipping_loss`: First or last syllable omitted due to PTT capture delay.

### Translation Evaluation Categories
- `semantic_meaning_error`: Translation reverses or alters core intent.
- `critical_operational_error`: Translation corrupts a critical operational term (e.g., translates *"tiền mặt"* [cash] to *"card"* or drops the fare amount).
- `omitted_information`: Non-critical detail present in source omitted in target.
- `added_information`: Unprompted detail added to target output.
- `acceptable_wording_variation`: Phrasing differs from reference translation but fully and accurately preserves operational intent.

### TTS & Audio Playback Categories
- `intelligibility`: Extent to which synthesized speech can be parsed by a listener from target seating position.
- `clipping_distortion`: Audio artifacts, clicks, or speaker saturation observed.
- `audibility`: Volume level relative to vehicle cabin background noise.
- `unexpected_output`: Mispronounced loan words, skipped punctuation, or unintended robotic accents.
- `synthesis_latency`: Delay between text submission and first audio buffer playback.

### Severity Levels
- `Critical`: Causes operational dispute, safety ambiguity, or complete interaction breakdown.
- `Major`: Requires user to repeat utterance or manually correct input; meaning degraded.
- `Minor`: Noticeable flaw or awkward phrasing; meaning remains understandable without confusion.
- `Informational`: Stylistic variation or harmless artifact.

---

## 8. Sampling Discipline & Population Claims

- **Exploratory Batch Sizing**: Initial spikes evaluate provisional batches of **10 to 25 fixtures** per scenario.
- **Purpose**: Exploratory batches serve to quickly uncover catastrophic failures, provider incompatibilities, and ergonomic blockers.
- **Statistical Boundary**: Small exploratory batches provide qualitative direction; they do **not** constitute statistical proof of general performance.
- **Prohibited Claims**: Engineers and agents must not generate speculative population claims, arbitrary confidence intervals, or unsupported market coverage assertions (e.g., *"99% accuracy"*, *"covers 95% of passengers"*). Any quantitative metric applies strictly to the evaluated sample under the documented conditions.

---

## 9. Controlled Comparison Rules

To attribute performance differences accurately across candidate engines:
1. **Single-Variable Variation**: When comparing candidate engines (e.g., Platform STT vs. Cloud STT), test using identical audio input, same hardware, and same playback conditions.
2. **Confounding Variable Prevention**: Do not simultaneously change acoustic environment, network profile, and candidate engine in a single comparison run.
3. **Fixture Consistency**: All competing candidates within a spike must evaluate against the exact same versioned fixture set (`dataset_id` and `dataset_version`).

---

## 10. Evidence Status Vocabulary

Experiment records and summary documents must classify findings using a standardized 4-state vocabulary:

| Status | Definition | Next Action |
| :--- | :--- | :--- |
| `exploratory` | Initial preliminary observation from a small run; not yet replicated. | Run replication test or expand fixture set. |
| `observed` | Documented empirical result from a controlled test run under recorded conditions. | Valid for candidate evaluation within documented scope. |
| `reproduced` | Confirmed through independent replication across multiple runs or test devices. | Strong evidence suitable for architectural decision gating. |
| `unresolved` | Inconsistent, contradictory, or irreproducible findings under seemingly identical conditions. | Block decision; isolate confounding variable in a follow-up spike. |

Arbitrary percentage-based pass/fail scoring systems and synthetic benchmark indices are prohibited until justified by production verification requirements.

---

## 11. Reproducibility Standard & Experiment Logging

Every spike test must produce a self-contained Markdown log under `docs/logs/` containing:
1. Complete metadata header (Fixture ID, device, OS, audio source, environment, provider, network, commit).
2. Per-fixture results table (Input, Expected Result, Observed Result, Latency, Error Category, Severity, Status).
3. Reviewer identification (Engineer/Agent).
4. Failure analysis notes detailing acoustic or semantic context.
5. Actionable verdict referencing the applicable Architecture Decision Gate.

---

## 12. Relationship to Specific Spikes

All subsequent exploratory spikes must directly adopt this protocol:

- **Spike-AUDIO-01 (PTT Ergonomics)**:
  - Uses Audio Metadata Schema.
  - Measures `clipping_loss` and button state timing against driver-authored fixtures.
- **Spike-STT-01 (Speech Recognition)**:
  - Uses Fixture Identity and STT Evaluation Categories.
  - Benchmarks candidate STT engines against identical audio fixtures under recorded acoustic conditions.
- **Spike-TRANS-01 (Translation Fidelity)**:
  - Uses Ground Truth Definitions (distinguishing exact wording from semantic equivalence).
  - Evaluates candidate translation options on critical operational phrases.
- **Spike-TTS-01 (Audio Audibility)**:
  - Uses TTS & Audio Playback Categories.
  - Tests rear-seat intelligibility from dashboard speaker across cabin noise scenarios.
- **Spike-LATENCY-01 (End-to-End Pipeline)**:
  - Uses Cumulative Timeline Schema across network metadata profiles.
  - Measures total turnaround across the candidate chain.
