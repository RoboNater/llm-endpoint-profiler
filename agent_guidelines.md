# AGENT_GUIDELINES.md

This document defines mandatory behavioral, architectural, and process rules for any automated agent (including Codex CLI) operating in this repository.

These guidelines are authoritative unless explicitly overridden by the human project owner.

---

## 1. Operating Principles

1. Always treat `specifications/SPEC.md` as the authoritative requirements document.
2. Never silently change requirements. If refinement is needed, explicitly document it.
3. Prefer clarity and determinism over cleverness.
4. Minimize unnecessary abstraction.
5. Avoid introducing new dependencies unless explicitly justified.
6. All changes must preserve ruff + mypy strict compliance.

---

## 2. Process Rules

1. Before beginning any substantial work, review:
   - specifications/SPEC.md
   - specifications/PROJECT_PLAN.md
   - PROJECT_STATUS.md

2. After completing any meaningful task, you MUST:
   - Update PROJECT_STATUS.md
   - Document what changed
   - List open design questions
   - Identify next recommended step

3. Stop at defined review gates and await human approval before proceeding.

4. Do not refactor large sections of code unless explicitly instructed.

5. Do not modify file structure without justification.

---

## 3. Architectural Rules

1. HTTP logic must remain isolated in `src/profiler/http_client.py`.
2. Data models must remain in `src/profiler/models.py`.
3. Probes must:
   - Live in `src/profiler/probes/`
   - Implement the shared probe interface
   - Return structured capability results
4. Report rendering must not contain probing logic.
5. CLI logic must remain in `src/profiler/main.py`.

---

## 4. Coding Standards

1. Python version: 3.13+
2. Full type hints required.
3. mypy strict mode compliance required.
4. ruff lint compliance required.
5. No global mutable state.
6. Prefer dataclasses for structured results.
7. All public functions must include docstrings.

---

## 5. Testing Rules

1. All new logic must include pytest coverage.
2. Probe logic should be testable without live network calls when possible.
3. Avoid coupling tests to specific external endpoints.

---

## 6. Repository Map

Authoritative specification:
- specifications/SPEC.md

Project plan:
- specifications/PROJECT_PLAN.md

Status tracking:
- PROJECT_STATUS.md

Core modules:
- src/profiler/models.py
- src/profiler/http_client.py
- src/profiler/report_generator.py
- src/profiler/probes/

---

## 7. Error Handling Philosophy

1. Distinguish clearly between:
   - Endpoint not implemented
   - Feature unsupported
   - Authentication failure
   - Network error
2. Never suppress unexpected exceptions silently.
3. Always record observed HTTP status codes in structured results.

---

## 8. Long-Term Vision

The architecture should allow evolution into:

- CI regression testing tool
- Endpoint certification system
- Compatibility diff engine
- Agent framework validation tool

Design for extensibility, but do not prematurely over-abstract.

---

## 9. Determinism Requirement

All outputs (especially reports) should be deterministic given identical inputs.
Avoid randomness unless explicitly requested.

---

End of AGENT_GUIDELINES.md

