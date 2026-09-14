# Engineering Logs & Debugging Records

This directory contains traceable, chronological records of all significant bugs, debugging investigations, benchmark experiments, failed spikes, and architectural post-mortems encountered during the lifecycle of the **Mtalk** project.

In accordance with our engineering principles:
- **Failures and dead ends must be documented**, not hidden.
- Engineering knowledge must outlive ephemeral AI conversations.
- Every major incident or non-trivial bug must have a root-cause analysis.

---

## Log Template

When logging a bug, debugging session, or spike result, create a new file in this directory with the naming convention `YYYY-MM-DD-<slug>.md` and follow this standard schema:

```markdown
# [Title: Short, descriptive summary of the issue or spike]

- **Date**: YYYY-MM-DD
- **Author**: [Human / Agent / Pair]
- **Component**: [e.g., Audio Pipeline / STT / ML Kit / Compose UI / CI]
- **Status**: [Resolved / Workaround / In-Progress / Abandoned]

---

### 1. Problem
Clear description of what went wrong or what question was being investigated.

### 2. Observation
What symptoms were observed? Include verbatim error messages, stack traces, logcat snippets, or benchmark metrics.

### 3. Reproduction
Exact steps or deterministic test case required to reproduce the failure.

### 4. Root Cause
Technical explanation of why the failure occurred at the system, OS, library, or hardware level.

### 5. Solution
The concrete fix or architecture adjustment applied to solve the root cause.

### 6. Trade-offs & Alternatives
What other solutions were considered? What are the performance, complexity, or maintenance costs of the chosen solution?

### 7. Verification
How was the fix verified? Which failure mode check was executed (Unit test, instrumented test, device test, or field test)?

### 8. Follow-up
Any preventive actions, lint checks, or architecture refinements needed to prevent recurrence.
```

---

## Log Directory Index

| Date | Title | Component | Status | Log File |
| :--- | :--- | :--- | :--- | :--- |
| *2026-09-14* | *Project Foundation Establishment* | *Repository Setup* | *Completed* | *Initialized* |
*(Future logs will be indexed here chronologically)*
