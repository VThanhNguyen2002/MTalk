# AGENTS.md — Mtalk Engineering Guidance for AI Agents

> **Project**: Mtalk — Translate for Drivers  
> **Repository Baseline**: Android 10+ (API 29+), Kotlin, Jetpack Compose  
> **Safety Rule**: [.agents/rules/safe-agent-workflow.md](file:///home/vietthanh/Projects/Projects/MTalk/.agents/rules/safe-agent-workflow.md) (Mandatory & Always Active)

---

## 1. Operating Principle

This project is an **agentic-assisted engineering project**. The primary objective is not merely to churn out code that compiles, but to practice and demonstrate disciplined software engineering:

- **Requirements Analysis & Scoping**
- **Evidence-Based Technology Selection** (Benchmarks, Spikes, official docs)
- **Architectural Integrity** ("Abstraction follows evidence")
- **Git Hygiene & Traceability**
- **Zero-Trust Security for Public Repositories**
- **Rigorous Verification & Testing**
- **Real-World Environmental Validation** (Noise, connectivity, cabin acoustics)

### Cardinal Rules for Agents:
1. **Code is not correct merely because it compiles.**
2. **An agent suggestion is not an architectural decision.**
3. **A green CI pipeline is not proof that the real-world product works.**
4. **Every architectural or dependency choice must be backed by evidence or an explicit Spike.**

---

## 2. Safety Rule Compliance

The workspace rule [.agents/rules/safe-agent-workflow.md](file:///home/vietthanh/Projects/Projects/MTalk/.agents/rules/safe-agent-workflow.md) is part of the engineering foundation.
- **Never** delete, replace, weaken, or rewrite this rule unless explicitly requested by the user.
- **Never** run destructive Git commands (`git reset --hard`, `git clean -fd`, etc.).
- **Never** modify `.git/` directly.
- **Never** expose secrets (`.env`, keystores, credentials, tokens).
- **Never** destroy uncommitted work. Run `git status` before initiating significant changes.
- **Always** report modified, created, and deleted files after significant actions.

---

## 3. Agentic Workflow Loop

All feature design and code modifications follow this disciplined feedback cycle:

```
Human Requirement
      ↓
Agent Research (Primary / Official Sources)
      ↓
Agent Challenge & Alternatives (Identify trade-offs & edge cases)
      ↓
Evidence (Spike, Benchmark, or Proof)
      ↓
Decision (Document in Project Docs)
      ↓
Implementation (Minimal, clean, provider-agnostic)
      ↓
Verification (Unit, UI, Integration, Device)
      ↓
Real-World Evaluation (Field testing, synthetic noise)
      ↓
Iteration
```

- **Challenge Assumptions**: If a requirement seems ambiguous, conflicting, or risks driver safety, stop and explain the conflict.
- **Propose Spikes**: If evidence is missing for an architectural choice (e.g., STT engine selection), propose an isolated Spike rather than guessing.

---

## 4. Context Management & Source of Truth

Agent context windows and chat histories are ephemeral and volatile.
- **Git and project documentation are the sole sources of truth.**
- The project must remain fully reproducible and understandable if the agent model, IDE, machine, or account changes.
- **Do not introduce premature indexing systems** (e.g., CodeGraph) until repository scale creates an evidence-backed navigation problem.

### Source-of-Truth Documents:
- [PROJECT_VISION.md](file:///home/vietthanh/Projects/Projects/MTalk/PROJECT_VISION.md): Product scope, user persona, problem statement, core pipeline, and UX direction.
- [ENGINEERING_RULES.md](file:///home/vietthanh/Projects/Projects/MTalk/ENGINEERING_RULES.md): Technical policies, decision classifications, verification matrix, security rules, CI/CD, Git standards.
- [README.md](file:///home/vietthanh/Projects/Projects/MTalk/README.md): Repository entrance, architecture overview, project status.
- [docs/logs/README.md](file:///home/vietthanh/Projects/Projects/MTalk/docs/logs/README.md): Debugging logs, spike outcomes, and failure investigations.

---

## 5. Verification Model: "Environment Must Match Failure Mode"

Before declaring any feature complete, agents must evaluate:

| Failure Mode | Verification Environment | Required Check |
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

## 6. Generated Code Verification Gate

Agent-generated code is **NOT** ready to merge until:
1. **Source is inspected** for unnecessary bloat or over-abstraction.
2. **`git diff` is reviewed** line by line.
3. **Build passes** via `./gradlew assembleDebug` (when application scaffold is established).
4. **Unit and UI tests pass** via `./gradlew test` (when application scaffold is established).
5. **Static checks and lint pass** via `./gradlew lint` (when application scaffold is established).
6. **Zero secrets check** confirms no keys, tokens, or PII were added.
7. **Verification matches failure mode** per the matrix above.
8. **Limitations and trade-offs are documented**.

---

## 7. Non-Goals for Foundation

Do NOT implement or scaffold:
- Premature backend infrastructure: MVP currently has no backend requirement. (A backend is not prohibited if evidence later justifies it, but must not be scaffolded prematurely).
- 100+ language support (focus on Vietnamese ↔ English first).
- Floating window overlays, background recording services, wake-word listeners, or accessibility hacks.
- Commercial billing, Google Play in-app purchases, or Play Store publishing flows.
- Premature abstractions: generic factory registries, complex event buses, dynamic plugin loaders.
