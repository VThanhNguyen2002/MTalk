# AUDIO-01 & STT-01 Execution Plan

> **Project**: MTalk — Translate for Drivers
> **Status**: Spike Phase 1 Execution Plan (Documentation Only)
> **Governing Documents**:
> - [docs/spikes/measurement-ground-truth-protocol.md](measurement-ground-truth-protocol.md)
> - [docs/spikes/fixtures/README.md](fixtures/README.md)
> - [ENGINEERING_RULES.md](../../ENGINEERING_RULES.md)
> - [PROJECT_VISION.md](../../PROJECT_VISION.md)

---

## 1. Purpose

This document is the concrete, pre-execution plan for two foundational spikes:

- **AUDIO-01**: Investigate push-to-talk audio capture ergonomics and recording fidelity on a real Android device.
- **STT-01**: Evaluate speech-to-text candidate options against driver-authored utterances under realistic acoustic conditions.

These spikes reduce the two largest unknowns before any application architecture is committed:

1. Can the audio capture interaction reliably obtain usable speech samples in a vehicle cabin environment?
2. Can a speech-to-text candidate accurately transcribe short, domain-specific Vietnamese and English utterances with enough fidelity to be translation-ready?

Neither spike selects a final provider. Neither spike introduces application code, dependencies, or production architecture. Both spikes produce documented empirical observations that inform subsequent architectural decisions.

---

## 2. Current Evidence Baseline

| Item | Status |
| :--- | :--- |
| Android scaffold | Complete (`05627d4`) |
| Day-1 CI | Active (GitHub Actions) |
| Spike discovery baseline | Complete (`f5946df`) |
| Measurement & Ground-Truth Protocol | Complete (`a88c9cb`) |
| Privacy-safe text fixtures (v1) | Complete (`63154c4`) |
| Fixture/protocol alignment | Complete (`1e1201d`) |
| Audio recordings | **Not yet created** |
| STT provider evaluation | **Not yet started** |

**Fixture set available**: `docs/spikes/fixtures/mtalk-v1-fixtures.json`
- 16 fixtures; 11 `vi-VN → en-US` (driver-authored), 5 `en-US → vi-VN` (synthetic)
- Criticality: 7 `critical`, 6 `operational`, 3 `normal`
- SHA-256 hashes verified; all fields aligned to measurement protocol schema

---

## 3. AUDIO-01

### Objective

AUDIO-01 investigates the following open questions:

1. Does press-and-hold PTT capture complete utterances, or does it clip opening or trailing syllables?
2. Does tap-to-start / tap-to-stop provide materially different clipping behavior?
3. Is ambient vehicle cabin noise captured at a level that interferes with speech content?
4. Are short utterances (3–6 words) captured reliably, including their boundaries?
5. Does the recording contain enough speech signal for a downstream STT candidate to work with?
6. Are there observable latency anomalies between the PTT interaction and actual capture start/stop?

AUDIO-01 does **not** select an audio API implementation for production. It produces observations that determine whether PTT (hold-to-speak) is the appropriate interaction model and what capture conditions should be assumed in STT-01.

### Inputs

**Fixture subset recommended**: Start with an exploratory batch of 10 fixtures drawn from the full v1 set. The selection is designed to cover all critical fixtures and sample the operational range for failure-mode discovery; it is not statistically representative of the full fixture population or of real-world driving scenarios. Select the subset as follows:

| Priority | Criterion | Target Count |
| :--- | :--- | :--- |
| Mandatory | All 7 `critical` fixtures | 7 |
| Fill-in | Short utterances (≤8 words) to surface clipping at start/end | 2 from `operational` |
| Fill-in | Longest utterance in the set (stress test boundary capture) | 1 |

This gives 10 fixtures for the first batch. If no clipping or boundary anomalies are found, the remaining 6 fixtures may optionally be added to confirm consistency.

**Fixture IDs for the initial batch**:
- All `critical` fixtures: `MTALK-VI-EN-001`, `MTALK-VI-EN-002`, `MTALK-VI-EN-003`, `MTALK-VI-EN-004`, `MTALK-EN-VI-001`, `MTALK-EN-VI-002`, `MTALK-EN-VI-003`
- Short utterance candidates from `operational`: `MTALK-VI-EN-011` (greeting), `MTALK-VI-EN-006` (traffic delay, ~15 words)
- Longest utterance in set: `MTALK-VI-EN-008` (request for repetition)

Each recording must reference the `fixture_id` and `source_hash_sha256` from the fixture manifest.

### Recording Conditions

The following conditions define the acoustic matrix for this exploratory batch. No specific dB values are claimed; conditions are described by reproducible labels.

| Condition ID | Label | Description |
| :--- | :--- | :--- |
| `COND-QUIET` | Quiet stationary (control) | Indoor quiet room, engine off, no fans, no traffic noise |
| `COND-CABIN-IDLE` | Stationary vehicle, engine idling | Vehicle parked, engine running, AC at normal setting, windows closed |
| `COND-CABIN-AMBIENT` | Stationary vehicle, exterior ambient | Vehicle parked, engine off or idling, windows partly open, typical urban ambient noise |

**Rationale**: Start with `COND-QUIET` to establish a clean baseline. Then proceed to cabin conditions. This separates interaction ergonomics from acoustic noise effects.

**Interaction models to compare** (experimental variants — all testing occurs with vehicle safely stationary; neither variant is automatically selected as the final MVP interaction; no phone interaction while driving is implied or validated by this experiment):
- **PTT-A**: Press-and-hold — touch down begins capture, touch up ends capture
- **PTT-B**: Tap-to-start / tap-to-stop — first tap begins capture, second tap ends capture

**Speaker positions** (controlled test conditions for this spike — not validated UX requirements or universal operating distances; usefulness may be revisited after real-device testing):
- **POS-HAND**: Device held naturally, approximately 30–50 cm from mouth
- **POS-DASH**: Device placed on dashboard mount, speaker approximately 60–80 cm away

POS-HAND is the primary position. POS-DASH is attempted for at least the 7 critical fixtures to surface any degradation from dashboard placement. These distances are the initial controlled condition for the spike; they are not claimed as optimal or as the definitive deployment distances.

**Utterance variations**:
- Normal speaking pace (primary)
- Faster pace (secondary, for 2–3 critical fixtures only)
- Intentional speech start immediately at PTT activation (stress test for opening syllable clipping)

### Recording Metadata

Every recording file must have a metadata record. This is **recording-session metadata**, distinct from the fixture definition.

**Minimum required per recording**:

```
recording_id:          Unique identifier, e.g. AUDIO-REC-001
fixture_id:            From mtalk-v1-fixtures.json
fixture_version:       From mtalk-v1-fixtures.json
source_hash_sha256:    From mtalk-v1-fixtures.json (confirms which utterance was spoken)
interaction_model:     PTT-A or PTT-B
speaker_position:      POS-HAND or POS-DASH
device_model:          OEM and model name (e.g., Samsung Galaxy A52)
android_version:       OS version and API level (e.g., Android 13 / API 33)
audio_source:          Hardware input used (e.g., device_built_in_bottom_mic)
recording_environment: COND-QUIET / COND-CABIN-IDLE / COND-CABIN-AMBIENT
speaker_language:      vi-VN or en-US
dialect_accent:        e.g., vi-Southern, en-US-American (required for audio fixtures)
recording_date:        ISO 8601 date (e.g., 2026-10-01)
recording_format:      Provisional test configuration for reproducible comparison, e.g. 16kHz_16bit_mono_PCM_WAV if
                       the capture tool allows it; device-default otherwise. This is not yet an MTalk
                       production audio contract and may be changed if AUDIO-01 or STT-01 evidence requires it.
notes:                 Any anomaly, retry reason, or observation during this recording
```

**Note**: `dialect_accent` is required here because this is an audio fixture; the protocol defers this field from text-only fixtures to the audio recording stage.

**Recording file naming convention** (proposed):
```
AUDIO-REC-{NNN}_{fixture_id}_{condition}_{interaction_model}_{position}.wav
Example: AUDIO-REC-001_MTALK-VI-EN-001_COND-QUIET_PTT-A_POS-HAND.wav
```

> [!CAUTION]
> Audio recording files are **not committed to the public Git repository**. They remain on local device storage or a private local directory only. Only the metadata records and experiment log (text Markdown) are committed.

### Failure Modes

| Failure Mode | How to Observe / Detect |
| :--- | :--- |
| `PERM-DENIED` | RECORD_AUDIO permission not granted; capture API returns error or silent signal |
| `MIC-UNAVAILABLE` | Microphone hardware busy; another app holds the audio source |
| `CAPTURE-START-FAIL` | Capture API throws exception or returns error code on start |
| `CAPTURE-STOP-FAIL` | Capture does not stop cleanly; recording continues beyond PTT release |
| `OPENING-CLIP` | First syllable of utterance audibly absent in playback |
| `TRAILING-CLIP` | Final word or syllable of utterance absent in playback |
| `SILENCE-PREPEND` | Perceptibly long silence before speech begins in recording |
| `SILENCE-APPEND` | Perceptibly long silence after speech ends in recording |
| `NOISE-DOMINATES` | Background noise level makes speech unintelligible on playback |
| `SPEECH-TOO-QUIET` | Speech signal substantially weaker than background noise |
| `WRONG-INPUT-SOURCE` | Recording appears to originate from a different microphone than expected |
| `UNEXPECTED-LATENCY` | Noticeable perceptual delay between PTT action and capture start; record as qualitative observation only. No numeric threshold applies — this is an exploratory observation marker, not a product requirement or acceptance criterion. |
| `LIFECYCLE-INTERRUPT` | App loses focus or capture interrupted by incoming call or notification |
| `DEVICE-SPECIFIC` | Behavior differs materially from expected; likely OEM-level variation |

For each failure mode, record: `recording_id`, `fixture_id`, condition, interaction model, and a written description of what was observed.

### Procedure

1. **Prepare**: Verify `RECORD_AUDIO` permission is granted on the test device.
2. **Control condition first**: Complete all selected fixture recordings under `COND-QUIET` before introducing vehicle acoustic conditions.
3. **PTT-A first, then PTT-B**: Complete the PTT-A batch for all selected fixtures before switching to PTT-B.
4. **Listen immediately**: Play back each recording immediately after capture; note any clipping or noise before moving to the next fixture.
5. **Repeat if clipping detected**: Re-record once in the same condition. Keep both takes; label the retry.
6. **Fill metadata immediately**: Complete all metadata fields before moving to the next fixture.
7. **Dashboard position subset**: After the primary POS-HAND batch, repeat all 7 critical fixtures with POS-DASH.

### Outputs

| Artifact | Format | Committed to Repo |
| :--- | :--- | :--- |
| Audio recording files | WAV or device-native format | **No** — local storage only |
| Per-recording metadata | Structured fields within experiment log | Yes |
| Experiment log | Markdown under `docs/logs/audio-01/` | Yes |
| Clipping/failure observation table | Per-fixture, per-condition | Yes |
| Verdict and evidence status | Per protocol vocabulary | Yes |

### Evidence Criteria

AUDIO-01 has produced usable evidence when:

- All 10 selected fixtures have at least one clean recording under `COND-QUIET`.
- Clipping behavior under `COND-CABIN-IDLE` is documented for all 7 critical fixtures using both PTT-A and PTT-B.
- All observed failure modes are recorded in the experiment log with fixture ID and condition.
- POS-DASH recordings exist for at least the 7 critical fixtures.
- Metadata is complete for every recording intended for STT-01 input.

### Architecture Revisit Conditions

The following findings would require revisiting the PTT interaction model before proceeding:

- Opening or trailing clipping is repeatedly observed on otherwise valid recordings in the control condition (`COND-QUIET`) and materially prevents reliable downstream STT evaluation. No numeric clipping rate threshold is claimed; the judgment is qualitative based on whether clean, complete recordings can be obtained for the critical fixtures.
- No condition exists under which a clean baseline recording can be obtained for the 7 critical fixtures.
- Microphone API is unavailable on the test device class.

If PTT-A and PTT-B produce materially different clipping rates, both sets are carried into STT-01 as separate audio inputs, explicitly labeled.

---

## 4. STT-01

### Objective

STT-01 investigates the following open questions:

1. Can candidate STT options transcribe short Vietnamese operational utterances with acceptable fidelity under stationary vehicle acoustic conditions?
2. Are critical terms (fare amounts, payment methods, destination terms) reliably recognized or systematically substituted?
3. How does transcription quality degrade from a clean recording condition to cabin noise conditions?
4. What is the observable latency from audio input to transcription result delivery?
5. How do candidates handle short utterances, ambiguous boundaries, and domain-specific vocabulary?
6. What failure modes (empty result, timeout, wrong language) are observed in practice?

STT-01 does **not** select a production STT provider. It produces evidence for Decision Gate D-STT (Section 9).

### Inputs

**Input source**: Audio recordings produced by AUDIO-01 under documented conditions.

STT-01 planning, fixture mapping, and evaluation rubric design proceed in parallel with AUDIO-01 preparation. STT-01 evaluation against real audio cannot begin until AUDIO-01 has produced clean baseline recordings for the selected fixture subset.

**Fixture mapping**: Every audio recording passed to an STT candidate maps 1:1 to a fixture via `fixture_id` and `source_hash_sha256`. The `source_utterance` field serves as the provisional `expected_transcription` unless AUDIO-01 reveals that the speaker's natural delivery differs from the written form, in which case `expected_transcription` must be declared explicitly and the fixture versioned.

**Starting subset**: The same 10 fixtures used in AUDIO-01.

### Evaluation Contract

The full evaluation pipeline per fixture:

```
fixture.source_utterance           (text prompt given to speaker)
        │
        ↓
AUDIO-01 recording                 (actual spoken audio, documented condition)
        │
        ↓
expected_transcription             (= source_utterance for current text-only fixtures;
        │                           explicit field required when audio delivery differs)
        ↓
STT candidate input                (audio file passed to STT API or library)
        │
        ↓
STT output string                  (candidate's verbatim transcription result)
        │
        ↓
Error analysis:
  Compare STT output vs. expected_transcription  → transcription accuracy
  Compare STT output vs. intended_meaning        → operational impact
  Per-term inspection against critical-term list → critical-term errors
        │
        ↓
Per-fixture verdict
```

**Key distinction** (protocol Section 4): A transcription differing in word choice but preserving `intended_meaning` may satisfy operational equivalence. A transcription that corrupts a critical term (e.g., `"năm mươi nghìn"` → `"năm mươi"`) constitutes a `CRITICAL-TERM-ERROR` regardless of overall sentence similarity. Both levels are recorded separately per fixture.

### Evaluation Categories

| Category | Definition |
| :--- | :--- |
| `CORRECT` | STT output matches expected_transcription exactly or is operationally equivalent |
| `SUBSTITUTION` | A word replaced by a different word that changes meaning |
| `INSERTION` | Extra words appear in output not present in source |
| `DELETION` | Words from source are absent from output |
| `CRITICAL-TERM-ERROR` | Distortion or omission of a term affecting payment, destination, or safety |
| `EMPTY-RESULT` | STT returns empty string, no result, or times out |
| `WRONG-LANGUAGE` | Output produced in an unexpected language |
| `PARTIAL` | Truncated output suggesting audio was cut short before result delivery |

Additionally record per fixture:
- **Operational impact**: `none` / `minor` / `significant` / `critical` (per protocol Section 7)
- **Latency (ms)**: Time from audio submission to final result delivery, where measurable

**Critical-term list** (derived from fixture content):

| Term | Fixture(s) |
| :--- | :--- |
| `năm mươi nghìn` | MTALK-VI-EN-002 |
| `tiền mặt`, `chuyển khoản` | MTALK-VI-EN-001 |
| `QR`, `taplo` | MTALK-VI-EN-004 |
| `sảnh B`, `cột số bốn` | MTALK-VI-EN-005 |
| `tấp vào lề`, `pull over` | MTALK-EN-VI-001 |
| `năm trăm nghìn`, `500,000` | MTALK-EN-VI-002 |
| `Terminal 2`, `Nhà ga số 2` | MTALK-EN-VI-003 |

These terms must be individually inspected even when aggregate transcription quality appears acceptable.

### Critical Phrase Handling

All 7 `critical` fixtures must be individually evaluated, not aggregated.

For each `critical` fixture:
1. Record the verbatim STT output string.
2. Identify every critical term present in `source_utterance`.
3. Confirm each critical term is correctly present in the STT output.
4. If any critical term is absent, substituted, or distorted: mark `CRITICAL-TERM-ERROR` for that fixture.
5. Record whether `intended_meaning` remains inferrable from the output, or is lost.

**Aggregate metrics are not sufficient**: A low overall word error rate does not establish that all 7 critical fixtures pass. Each must be individually inspected.

### Comparison Design

When evaluating multiple STT candidates:

- All candidates receive the **same audio file** for each fixture.
- All candidates are evaluated against the **same `expected_transcription`**.
- Device, OS version, and network condition are documented and held constant within a comparison run.
- Variables that may be independently varied across separate runs (one variable at a time):
  - Acoustic condition (`COND-QUIET` vs. `COND-CABIN-IDLE`)
  - Speaker position (`POS-HAND` vs. `POS-DASH`)
  - STT candidate
  - Network condition (if candidate is network-dependent)

Do not vary multiple independent variables in the same comparison run.

### Failure Modes

**Infrastructure / Experiment Failures** (do not indicate STT quality):

| Failure Mode | Description |
| :--- | :--- |
| `PERM-DENIED` | Permission denied; resolve before running |
| `PROVIDER-INIT-FAIL` | STT API or library fails to initialize; document error and retry once |
| `TIMEOUT` | No result returned within documented wait period; record elapsed time, mark `EMPTY-RESULT (timeout)` |
| `NETWORK-UNAVAILABLE` | Network-dependent candidate has no connectivity; test as a deliberate offline condition |
| `RATE-LIMIT` | API quota exhausted; pause and resume; document in log |

**STT Quality Failures** (indicate candidate limitations):

| Failure Mode | Description |
| :--- | :--- |
| `EMPTY-RESULT` | No transcription returned for a non-empty audio input |
| `WRONG-LANGUAGE` | Output produced in wrong language |
| `CRITICAL-TERM-ERROR` | Critical term distorted or missing |
| `DOMAIN-VOCAB-MISS` | Domain-specific words (e.g., `taplo`, `chuyển khoản`) unrecognized |
| `SHORT-UTTERANCE-MISS` | Utterances of ≤4 words consistently fail or return empty |

Both categories must be separated in the experiment log. Infrastructure failures do not count as STT quality evidence.

### Procedure

1. **Pre-condition**: AUDIO-01 must have produced at least one clean recording per selected fixture under `COND-QUIET`.
2. **Baseline candidate first**: Evaluate the simplest available candidate first (e.g., platform API) to establish a reference before evaluating alternatives.
3. **Control condition first**: Evaluate all candidates against `COND-QUIET` recordings before introducing cabin noise recordings.
4. **Record verbatim**: Capture STT output exactly as returned. Do not post-process, spell-correct, or paraphrase.
5. **Time the response**: Note elapsed time from audio submission to final text delivery for each run.
6. **Inspect critical terms immediately**: After each run, check the critical-term list for the current fixture before proceeding.
7. **Document failure modes as they occur**: Do not reconstruct from memory after the session.
8. **Network-dependent candidates**: Document network condition per run; include at least one isolated offline or throttled-network run if the candidate requires connectivity.

### Outputs

| Artifact | Format | Committed to Repo |
| :--- | :--- | :--- |
| Experiment log (per candidate) | Markdown under `docs/logs/stt-01/` | Yes |
| Per-fixture results table | fixture_id, STT output, categories, latency, verdict | Yes |
| Critical-term inspection table | Per critical fixture, per critical term | Yes |
| Provider metadata | API name, configuration, model version if accessible | Yes |
| Evidence status per fixture/candidate | Per protocol vocabulary | Yes |
| Failure log | Infrastructure and quality failures separated | Yes |

### Evidence Criteria

STT-01 has produced usable evidence when:

- At least one STT candidate has been evaluated against all 10 selected fixtures under `COND-QUIET`.
- All 7 critical fixtures have individual results and critical-term inspection records.
- All failure modes encountered are documented in the experiment log.
- Evidence status is assigned per fixture per candidate (per protocol vocabulary: `exploratory`, `observed`, `reproduced`, `unresolved`).
- Latency observations (not thresholds) are recorded for each candidate.

### Architecture Revisit Conditions

The following findings would require revisiting STT provider direction before implementing any production code:

- Critical-term transcription failures are repeatedly observed across multiple critical fixtures under `COND-QUIET`, and the failure pattern is sufficient to indicate that the evaluated approach cannot reliably support the operational scenarios under test. No numeric count of failures is defined as the trigger; the judgment is based on whether the pattern of failure is consistent, reproducible, and pertains to terms that are operationally meaningful.
- The STT candidate does not demonstrate sufficiently consistent usable transcription across the evaluated `vi-VN` fixtures for the scenarios under test. Results remain exploratory and are not statistically representative; the judgment is based on whether enough fixtures produce output that is usable for downstream evaluation.
- Observed latency across all candidates makes the target interaction model untenable (assessed qualitatively against acceptable conversational pacing).

---

## 5. AUDIO → STT Dependency

```
Text fixture (source_utterance)
        │
        │  AUDIO-01: speaker reads fixture, PTT interaction records audio
        ↓
Audio recording file
        │
        │  expected_transcription = source_utterance for text-only fixtures;
        │  becomes an explicit field if recorded delivery differs (fixture versioned)
        ↓
Expected transcription (reference ground truth for STT evaluation)
        │
        │  STT-01: audio file submitted to STT candidate
        ↓
STT output string
        │
        │  Error analysis: STT output vs. expected_transcription (accuracy)
        │                  STT output vs. intended_meaning         (operational impact)
        ↓
Per-fixture observations (by candidate, by condition)
        │
        ↓
Evidence for Decision Gate D-STT
```

**Sequential dependency**: STT-01 evaluation against acoustic recordings requires audio files from AUDIO-01.

**Parallel work** (can proceed before AUDIO-01 completes):
- Evaluation rubric (this document)
- Fixture-to-expected-transcription mapping review
- STT candidate API initialization testing (without real audio)
- Experiment log template preparation

**What AUDIO-01 must deliver to STT-01**:
- Audio files (local, not committed): one or more recordings per fixture with documented condition
- Metadata records (committed as part of AUDIO-01 log): complete per-recording entries
- Clipping assessment: confirmation that each audio file used in STT evaluation is non-clipped

AUDIO-01 files with confirmed clipping must not be used as primary STT-01 input. They may be included as an explicitly labeled secondary stress-test run.

---

## 6. Real-Device Testing Boundary

| Category | Description |
| :--- | :--- |
| **Controlled spike experiment** | Single device, documented OS, documented condition, documented PTT model. Results apply to that specific device. |
| **Real-device exploratory observation** | Qualitative notes on ergonomics or unexpected behaviors during spike execution. |
| **Later user validation** | Multi-device, multi-environment, field testing with actual vehicle scenarios. Explicitly deferred to a future phase. |

**Device-level constraints**:
- All results must record `device_model`, `android_version`, and `android_api_level`.
- Results from one device cannot be generalized to the Android ecosystem without additional device-level replication.
- OEM-specific microphone behavior, audio routing quirks, or platform SpeechRecognizer service variations must be noted as device-specific observations rather than platform-wide conclusions.

---

## 7. Network and Latency Measurement

### Latency

For AUDIO-01:
- **PTT activation → capture start**: Observable delay between interaction and recording beginning (relevant to opening-clip risk).
- **PTT release → capture stop**: Observable delay between releasing PTT and recording ending.

No numeric target is established. Observations are qualitative or coarse-grained.

For STT-01:
- **Audio submission → first partial result** (if the candidate emits partial results)
- **Audio submission → final result delivery**

Measured as elapsed times per run, per candidate. No aggregate latency target is established. Full pipeline latency decomposition belongs to Spike-LATENCY-01, which cannot run until candidate components are selected.

### Network

STT candidates requiring network connectivity must be evaluated under documented conditions:

- **Primary**: Stable WiFi or strong cellular (establish documented baseline first)
- **Secondary**: Simulated degraded connectivity (airplane-mode or Android Developer Options throttle), if the candidate is network-dependent

Conditions are described by mechanism (airplane-mode, throttle setting name), not by claimed bandwidth or latency values. Network-independent candidates require only confirmation that no network calls are made during evaluation.

---

## 8. Privacy Boundary

### Permitted
- Audio recorded by the development team or consenting volunteer reading a fixture prompt aloud
- Session metadata describing the recording (device, condition, date, speaker language, dialect)
- STT output strings evaluated against fixture-sourced audio

### Prohibited
- Passenger audio recordings of any kind
- Live ride transcript fragments
- Recordings of real driver-passenger interactions, even anonymized
- Recordings where a non-consenting third party's voice is captured

### Repository boundary
- Audio recording files are **never committed** to the public repository
- Only experiment log Markdown files (text metadata, transcription strings) are committed
- No API keys, tokens, or STT provider credentials in committed files; injected via `local.properties` or environment variables

> [!CAUTION]
> If any recording accidentally captures a non-consenting third party, that recording must not be used as a spike input and must not be submitted to an STT provider.

---

## 9. Decision Gates

| Gate ID | Decision | Evidence Required Before Deciding |
| :--- | :--- | :--- |
| **D-PTT** | PTT model: hold-to-speak vs. tap-to-start/tap-to-stop | AUDIO-01 clipping rate and ergonomic observations across both models and conditions |
| **D-AUDIO-API** | Android audio capture API (AudioRecord / MediaRecorder / other) | AUDIO-01 capture reliability and format observations |
| **D-STT** | STT provider for the MVP pipeline | STT-01 transcription quality on critical fixtures, latency observations, failure behavior, on-device vs. network dependency |
| **D-OFFLINE** | On-device-only vs. network-dependent STT architecture | STT-01 comparative results across on-device and network-dependent candidates; Spike-HYBRID-01 if fallback is needed |

No gate decision is made in this document. All four gates remain open pending spike evidence.

---

## 10. Explicitly Deferred Decisions

| Item | Deferral Reason |
| :--- | :--- |
| Final STT provider selection | Requires STT-01 evidence |
| Production audio capture code | Requires D-PTT and D-AUDIO-API decisions |
| Translation engine evaluation | Spike-TRANS-01; proceeds independently |
| TTS engine evaluation | Spike-TTS-01; proceeds independently |
| End-to-end latency target | Requires candidate pipeline; Spike-LATENCY-01 |
| Provider abstraction classes | Abstraction follows evidence; not designed before D-STT resolves |
| DI framework selection | Not required until production components exist |
| Backend architecture | MVP baseline has no backend requirement |
| Multi-language support | Scoped to vi-VN ↔ en-US for the current spike phase |
| Confidence-based fallback | Spike-HYBRID-01; requires STT candidate output first |
| Statistical significance claims | Requires a larger fixture set; current batch is exploratory only |
