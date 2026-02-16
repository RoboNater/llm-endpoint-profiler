# LLM Endpoint Capability Profiler - Project Plan

## 1. Planning Baseline

This plan is derived from `specifications/SPEC.md` and assumes:

- Python `3.13+`
- `uv` for environment and task execution
- `ruff` for linting
- `mypy` in strict mode
- `pytest` for testing
- `httpx` as the HTTP client
- Fully type-hinted, modular probe-based architecture
- Minimal token and request usage during probing

The plan is review-gated. After each phase:

1. Update `specifications/SPEC.md` if refinements were discovered.
2. Update `specifications/PROJECT_PLAN.md` with completed work and residuals.
3. Record open design questions.
4. Stop and await human review before starting the next phase.

---

## 2. Proposed Target Architecture (Implementation Direction)

### 2.1 Module Layout

Existing scaffold files are retained and expanded:

- `src/profiler/config.py`: CLI argument schema and runtime config model.
- `src/profiler/http_client.py`: typed wrapper around `httpx` with normalized errors.
- `src/profiler/models.py`: core capability models and error classification enums.
- `src/profiler/probes/base_probe.py`: probe protocol/ABC and probe result contract.
- `src/profiler/probes/models_probe.py`: `/v1/models` endpoint probe.
- `src/profiler/probes/chat_probe.py`: `/v1/chat/completions` probe and feature probes.
- `src/profiler/probes/responses_probe.py`: `/v1/responses` probe and feature probes.
- `src/profiler/probes/embeddings_probe.py`: `/v1/embeddings` probe.
- `src/profiler/report_generator.py`: Markdown + JSON rendering from capability model only.
- `src/profiler/main.py`: orchestration entrypoint for probe execution and report generation.

Recommended additions in later phases:

- `src/profiler/error_classification.py`: centralized heuristics and classifier helpers.
- `src/profiler/probes/assistants_probe.py`: `/v1/assistants` probe.
- `src/profiler/probe_registry.py`: probe ordering by mode (`safe|deep|exhaustive`).
- `src/profiler/diff_report.py` (optional): compare two JSON capability snapshots.
- `tests/fixtures/*.json`: canonical API response/error fixture files.

### 2.2 Capability Model Sketch

Core entities to standardize early:

- `CapabilityReport`
  - `target: TargetInfo`
  - `summary: ProbeSummary`
  - `endpoints: list[EndpointCapability]`
  - `models: list[ModelCapability]`
  - `generated_at: datetime`
- `EndpointCapability`
  - `name: str` (example: `"chat_completions"`)
  - `path: str` (example: `"/v1/chat/completions"`)
  - `supported: bool`
  - `http_status: int | None`
  - `classification: ErrorClass | None`
  - `features: list[FeatureCapability]`
  - `notes: str | None`
- `FeatureCapability`
  - `name: str` (example: `"streaming"`, `"tools"`, `"json_mode"`)
  - `supported: bool | None` (`None` when indeterminate)
  - `http_status: int | None`
  - `classification: ErrorClass | None`
  - `evidence: str | None` (short, sanitized)
- `ModelCapability`
  - `id: str`
  - `inferred_family: str | None`
  - `usable_for: list[str]` (example: `["chat", "embeddings"]`)

Suggested error classes:

- `ENDPOINT_NOT_FOUND`
- `METHOD_NOT_ALLOWED`
- `AUTH_REQUIRED`
- `AUTH_INVALID`
- `RATE_LIMITED`
- `FEATURE_UNSUPPORTED`
- `INVALID_REQUEST_SHAPE`
- `MODEL_NOT_FOUND`
- `SERVER_ERROR`
- `NETWORK_ERROR`
- `TIMEOUT`
- `UNKNOWN`

---

## 3. Phase Plan

## Phase 1 - Architecture Refinement

### Goals

- Lock core architecture decisions before coding probes deeply.
- Refine ambiguities in `specifications/SPEC.md`.
- Define strict typed contracts for capability model and probe interface.

### Deliverables (file-level)

- Update `specifications/SPEC.md` with clarified definitions:
  - What qualifies as endpoint support vs feature support.
  - Deterministic probe ordering and minimal request philosophy.
  - Required fields for Markdown and JSON output.
- Replace placeholder content in:
  - `src/profiler/models.py` with typed dataclasses/enums for capability model.
  - `src/profiler/probes/base_probe.py` with probe protocol/ABC and result contract.
  - `src/profiler/config.py` with CLI schema (`--base-url`, `--api-key`, `--output`, `--json`, `--timeout`, `--mode`).
- Add `src/profiler/error_classification.py` with initial status/error taxonomy.
- Add architecture notes in `docs/architecture.md`.

### Validation / Acceptance Criteria

- `SPEC.md` explicitly distinguishes:
  - endpoint missing (`404`) vs feature unsupported (`400`/validation-style failures).
  - auth failures (`401`/`403`) from unsupported capabilities.
- Probe interface contract exists and is type-checkable in strict mode.
- Capability model supports both Markdown and JSON reporting without ad hoc fields.
- CLI argument contract covers all FR/NFR-required flags.

### Test Strategy

- Add model and classifier unit tests:
  - `tests/test_models.py`
  - `tests/test_error_classification.py`
  - `tests/test_config.py`
- Include table-driven classification tests (status + payload -> `ErrorClass`).

### Documentation Updates

- `specifications/SPEC.md` (refinements and ambiguity resolution).
- `docs/architecture.md` (module responsibilities and data flow).
- `README.md` (CLI argument contract, no deep examples yet).

### Estimated Iterations

- `1-2` iterations.

### Review Gate (mandatory stop)

- Update `SPEC.md` and `PROJECT_PLAN.md` with outcomes.
- Record unresolved questions.
- Pause for human review.

---

## Phase 2 - Minimal Proof of Concept

### Goals

- Deliver minimal end-to-end profiler path with two endpoints and basic report.

### Deliverables (file-level)

- Implement typed HTTP wrapper in `src/profiler/http_client.py`:
  - Request helpers (`get`, `post`) with timeout and auth headers.
  - Normalized exception mapping (network/timeout/http).
- Implement probes:
  - `src/profiler/probes/models_probe.py` for `/v1/models`.
  - `src/profiler/probes/chat_probe.py` for `/v1/chat/completions` baseline detection.
- Implement orchestration in `src/profiler/main.py`:
  - run minimal probe set and aggregate into `CapabilityReport`.
- Implement Markdown report generation in `src/profiler/report_generator.py`.
- Wire CLI execution in `main.py` (repository root entrypoint stays minimal delegator).

### Validation / Acceptance Criteria

- Running the CLI against a target endpoint produces:
  - Markdown report file with endpoint support summary.
  - Clear classification examples for:
    - endpoint missing (`404`)
    - endpoint exists but invalid request (`400`)
- `/v1/models` and `/v1/chat/completions` detection produce typed capability entries.
- No unhandled exceptions escape CLI for expected probe failures.

### Test Strategy

- Add unit tests with mocked `httpx` transport:
  - `tests/test_http_client.py`
  - `tests/test_models_probe.py`
  - `tests/test_chat_probe.py`
  - `tests/test_report_generator_markdown.py`
- Add one integration-like CLI smoke test with monkeypatched client:
  - `tests/test_cli_poc.py`

### Documentation Updates

- Update `README.md` with POC usage and sample command.
- Add `examples/example_report.md` generated from deterministic fixture.
- Refine `SPEC.md` only if POC reveals conflicts.

### Estimated Iterations

- `2-3` iterations.

### Review Gate (mandatory stop)

- Update `SPEC.md` and `PROJECT_PLAN.md` with outcomes and changes.
- Record open questions.
- Pause for human review.

---

## Phase 3 - Prototype Expansion

### Goals

- Expand endpoint coverage and feature probing while preserving low-cost probe behavior.

### Deliverables (file-level)

- Add endpoint probes:
  - `src/profiler/probes/responses_probe.py`
  - `src/profiler/probes/embeddings_probe.py`
  - `src/profiler/probes/assistants_probe.py` (new)
- Expand feature probes for supported surfaces:
  - streaming
  - tools/function calling
  - JSON mode / structured output
  - logprobs
  - vision/multimodal
  - seed
- Implement JSON output serialization in `src/profiler/report_generator.py`.
- Ensure internal capability model in `src/profiler/models.py` captures endpoint + feature + model dimensions cleanly.
- Add `src/profiler/probe_registry.py` to control probe ordering and skip policy.

### Validation / Acceptance Criteria

- JSON output is deterministic and schema-stable for CI use.
- At least three surfaces have feature-level probing with explicit classifications.
- Probe execution avoids unnecessary token use:
  - shortest valid prompts
  - minimal `max_tokens`
  - minimal endpoint call count per feature
- Indeterminate states are represented explicitly (not inferred as unsupported).

### Test Strategy

- Add tests:
  - `tests/test_responses_probe.py`
  - `tests/test_embeddings_probe.py`
  - `tests/test_assistants_probe.py`
  - `tests/test_feature_probing.py`
  - `tests/test_report_generator_json.py`
- Add fixture-driven tests under `tests/fixtures/` for representative provider error payloads.

### Documentation Updates

- Extend `README.md` with JSON output examples.
- Add `docs/probes.md` describing probe logic and minimal payload choices.
- Update `SPEC.md` if new endpoint semantics require clarifications.

### Estimated Iterations

- `2-4` iterations.

### Review Gate (mandatory stop)

- Update `SPEC.md` and `PROJECT_PLAN.md` with outcomes and changes.
- Record open questions.
- Pause for human review.

---

## Phase 4 - Maturation

### Goals

- Improve reliability, explainability, and operational usefulness.

### Deliverables (file-level)

- Improve heuristics in `src/profiler/error_classification.py`:
  - Parse provider-style error payloads/messages conservatively.
  - Add confidence or evidence notes for ambiguous classifications.
- Add CLI modes in `src/profiler/config.py` and `src/profiler/main.py`:
  - `safe`: minimal endpoint existence checks.
  - `deep`: selected feature probes.
  - `exhaustive`: maximal supported probes.
- Optional diff capability:
  - `src/profiler/diff_report.py`
  - CLI flags for comparing two JSON reports.
- Improve logging/traceability:
  - structured probe trace in `src/profiler/main.py` or dedicated logger helper.

### Validation / Acceptance Criteria

- CLI mode behavior is deterministic and documented.
- Classification false-positives are reduced on fixture corpus.
- Probe traces make each decision auditable (request intent -> response -> classification).
- Optional diff output is meaningful and ignores non-semantic changes (timestamps).

### Test Strategy

- Add tests:
  - `tests/test_probe_modes.py`
  - `tests/test_error_classification_heuristics.py`
  - `tests/test_diff_report.py` (if implemented)
  - `tests/test_trace_logging.py`
- Expand fixture corpus to include auth errors, schema errors, throttling, server faults.

### Documentation Updates

- Update `README.md` with mode guidance and operational trade-offs.
- Add `docs/error-classification.md` for heuristic rationale and limitations.
- Refine `SPEC.md` with finalized mode behavior and classification semantics.

### Estimated Iterations

- `2-3` iterations.

### Review Gate (mandatory stop)

- Update `SPEC.md` and `PROJECT_PLAN.md` with outcomes and changes.
- Record open questions.
- Pause for human review.

---

## Phase 5 - Finalization and Documentation

### Goals

- Prepare for stable handoff and packaging.

### Deliverables (file-level)

- Final refinement pass on `specifications/SPEC.md`.
- Finalize `README.md` usage examples for common workflows.
- Finalize `examples/example_report.md` and add sample JSON report in `examples/`.
- Ensure packaging/entrypoints are complete in `pyproject.toml`.
- Confirm all modules are fully typed and lint/type/test clean.

### Validation / Acceptance Criteria

- `ruff check .` passes.
- `mypy --strict src` passes (or strict equivalent configured in project settings).
- `pytest` passes with meaningful probe and classification coverage.
- CLI is installable and runnable via `uv run`.
- Documentation reflects actual behavior and output fields.

### Test Strategy

- Full regression run in CI-like sequence:
  - lint
  - type-check
  - unit tests
  - deterministic report snapshot tests
- Add final contract tests asserting report schema stability.

### Documentation Updates

- `README.md`: complete setup, usage, modes, examples.
- `specifications/SPEC.md`: finalized accepted behavior.
- `docs/`: architecture, probes, classification, and troubleshooting.

### Estimated Iterations

- `1-2` iterations.

### Review Gate (mandatory stop)

- Update `SPEC.md` and `PROJECT_PLAN.md` with final outcomes.
- Record any deferred items.
- Pause for human sign-off.

---

## 4. Cross-Phase Risk Register

1. False classification due to provider-specific error shapes.
Mitigation: fixture corpus from multiple providers; conservative fallback to `UNKNOWN`.

2. Token overuse from feature probes.
Mitigation: strict minimal payload templates; mode-based depth controls; reuse endpoint-level evidence.

3. Coupling between probes and rendering.
Mitigation: enforce model-first design; rendering consumes only `CapabilityReport`.

4. Ambiguous auth vs capability failures.
Mitigation: classify auth errors before capability inference; short-circuit dependent probes.

5. Non-deterministic JSON output breaks diffs/CI.
Mitigation: stable ordering and explicit schema contract tests.

6. Strict typing friction slows delivery.
Mitigation: define stable typed contracts in Phase 1; avoid dynamic dict-driven internals.

---

## 5. Open Design Questions (to resolve during gates)

1. Should probe requests use a configurable model hint when `/v1/models` is unavailable?
2. What minimum evidence should be persisted for each capability without leaking sensitive payloads?
3. When should a feature be `unsupported` versus `indeterminate`?
4. Should auth failures abort all probes or continue with best-effort endpoint checks?
5. Is assistants probing required for MVP or optional until Phase 3 completion?
6. Should JSON output include raw response fragments for debugging, and if so, how are they redacted?
7. What is the stable schema/versioning strategy for JSON reports?

---

## 6. Suggested Execution Order

1. Complete Phase 1 and stop for review.
2. Complete Phase 2 and stop for review.
3. Complete Phase 3 and stop for review.
4. Complete Phase 4 and stop for review.
5. Complete Phase 5 and stop for final sign-off.

No implementation should proceed past a gate without explicit human approval.
