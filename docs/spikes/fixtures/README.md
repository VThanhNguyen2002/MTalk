# MTalk Evaluation Fixtures (v1)

> **Status**: Exploratory Baseline Fixture Set (v1.0.0)<br>
> **Governing Protocol**: [docs/spikes/measurement-ground-truth-protocol.md](../measurement-ground-truth-protocol.md)<br>
> **Manifest**: [mtalk-v1-fixtures.json](mtalk-v1-fixtures.json)

---

## 1. Purpose

This directory contains the initial, versioned, privacy-safe text fixtures for MTalk exploratory spikes. It establishes authoritative textual ground truth for:
- **Spike-AUDIO-01**: Utterance prompts for acoustic capture and PTT clipping tests.
- **Spike-STT-01**: Reference transcripts for speech-to-text accuracy evaluation.
- **Spike-TRANS-01**: Source sentences and expected translations for translation fidelity.
- **Spike-TTS-01**: English text inputs for in-cabin audibility and intelligibility tests.

---

## 2. Provenance & Privacy Boundary

All fixtures in this manifest are strictly **driver-authored** or **synthetic**:
- **Driver-authored (11 fixtures)**: Realistic operational phrases created from driver workflows (Vietnamese -> English).
- **Synthetic (5 fixtures)**: Structured simulated phrases representing passenger responses and requests (English -> Vietnamese).

> [!CAUTION]
> **Strict Privacy Rule**:
> - NO live passenger audio recordings.
> - NO live ride transcripts.
> - NO customer personal identifying information (PII).
> - NO uncontrolled third-party conversations.
>
> Unconsented third-party data must never be committed to this directory or any part of the MTalk repository.

---

## 3. Exploratory Status & Sampling

This is an **exploratory v1 fixture set** (16 fixtures), designed to exercise core driver-passenger coordination scenarios (fare payment, QR scanning, cash denominations, gate drop-offs, pickup terminal clarification, urgent stops).

- **Scope Limit**: These fixtures provide directional qualitative evidence for architectural exploration. They do **not** claim statistical representation of the entire riding population.
- **Criticality Classification**:
  - `critical`: High-cost operational interactions (fares, cash denominations, drop-off gates, airport terminals, urgent stops) requiring individual pass/fail tracking.
  - `operational`: Routine ride coordination (traffic delays, waiting, luggage loading, belongings reminders).
  - `normal`: Courtesies and comfort checks (greetings, AC comfort, gratitude).

---

## 4. How Spikes Reference These Fixtures

Future spike experiments and evaluation logs under `docs/logs/` must:
1. Reference the exact `fixture_id` (e.g., `MTALK-VI-EN-001`) and `fixture_version` (`1.0.0`).
2. Record `source_hash_sha256` to confirm input integrity.
3. Compare observed output against `intended_meaning` and `expected_translation` according to the [Measurement & Ground-Truth Protocol](../measurement-ground-truth-protocol.md).
4. Any proposed modification or addition to ground-truth text requires incrementing the manifest version.
