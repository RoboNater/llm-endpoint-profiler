# LLM Endpoint Capability Profiler

High-level functional and non-functional requirements for the project.

---

## 1. Introduction

### 1.1 Purpose

This specification defines the functional scope, external interfaces, design constraints, and validation criteria for the **LLM Endpoint Capability Profiler** — a Python-based CLI and library that discovers and documents the capabilities of arbitrary OpenAI-compatible LLM API endpoints.

This document is the authoritative definition of what must be built. It is intentionally implementation-agnostic and will guide Codex CLI planning and iterative development.

---

## 2. Product Overview

### 2.1 Problem Statement

LLM providers expose varying subsets of OpenAI-style APIs (Chat Completions, Responses API, Embeddings, Assistants, etc.). Many endpoints claim compatibility but differ in:

- Supported routes
- Supported request parameters
- Feature support (streaming, tools, JSON mode, vision)
- Model availability

Developers currently must manually probe endpoints to determine compatibility.

### 2.2 Goal

Provide an automated tool that:

- Programmatically discovers supported API surfaces
- Probes feature-level capability support
- Enumerates available models
- Distinguishes unsupported endpoints from unsupported features
- Generates a structured Markdown capability report
- Optionally generates machine-readable JSON output

---

## 3. Scope

The profiler SHALL:

- Accept API base URL and authentication credentials
- Attempt discovery of standard OpenAI-compatible endpoints, including but not limited to:
  - /v1/models
  - /v1/chat/completions
  - /v1/responses
  - /v1/embeddings
  - /v1/assistants
- Perform minimal, low-cost probe requests
- Detect feature support such as:
  - Streaming
  - Tool calling
  - JSON mode / structured output
  - Logprobs
  - Vision / multimodal inputs
  - Seed parameter
- Generate structured reports

The profiler SHALL NOT (at this stage):

- Perform benchmarking or load testing
- Act as a proxy or middleware
- Attempt to reverse-engineer undocumented proprietary APIs

---

## 4. Requirements

### 4.1 Functional Requirements

FR1 — The application SHALL accept a base API URL and authentication credentials.

FR2 — The application SHALL detect which known API surfaces are implemented.

FR3 — The application SHALL perform minimal valid requests to confirm feature support.

FR4 — The application SHALL distinguish:
- Endpoint not implemented (e.g., HTTP 404)
- Endpoint exists but feature unsupported (e.g., HTTP 400 with parameter error)
- Authentication errors

FR5 — The application SHALL enumerate available models.

FR6 — The application SHALL generate a Markdown capability report.

FR7 — The application SHALL generate a JSON capability file.

FR8 — The CLI SHALL handle and document errors gracefully.

---

### 4.2 Non-Functional Requirements

NFR1 — Implementation SHALL use Python 3.13+ and be compatible with uv-managed environments.

NFR2 — All code SHALL use type hints.

NFR3 — HTTP interactions SHALL be abstracted behind a client wrapper.

NFR4 — The system SHALL minimize token usage and avoid unnecessary requests.

NFR5 — The probe architecture SHALL be modular and extensible.

NFR6 — The project SHALL include automated tests using pytest.

NFR7 — The project SHALL enforce code quality standards using ruff for linting and mypy for static type checking.

NFR8 — Type checking SHALL be configured in strict mode unless explicitly justified.

NFR9 — The design SHALL separate:
- Capability discovery logic
- Data modeling
- Report rendering

---

## 5. External Interfaces

### 5.1 Command Line Interface

Example usage:

    profiler \
      --base-url https://custom.llm.api \
      --api-key sk-xxxxxxxx \
      --output report.md \
      --json report.json

Required parameters:

- --base-url
- --api-key

Optional parameters:

- --output
- --json
- --timeout
- --mode (safe | deep | exhaustive)

---

### 5.2 Network Interface

The profiler SHALL communicate over HTTPS using JSON payloads consistent with OpenAI-style APIs.

All HTTP communication SHALL be encapsulated within a dedicated HTTP client abstraction.

---

## 6. Architecture Constraints

The system SHALL implement a probe abstraction:

Each probe MUST:

- Declare which endpoint it tests
- Execute a minimal probe request
- Interpret the HTTP response
- Return structured capability results

Report generation MUST operate only on structured capability objects.

---

## 7. Capability Data Model

The profiler SHALL internally represent:

- Endpoint-level capability
- Feature-level capability
- Model-level capability

Each capability SHALL include:

- Supported (boolean)
- Status code observed
- Error classification
- Optional notes

---

## 8. Validation & Acceptance Criteria

Proof-of-Concept is complete when:

- /v1/models detection works
- /v1/chat/completions detection works
- A Markdown report is generated

Prototype is complete when:

- Modular probe system is implemented
- JSON output is generated
- Feature probing implemented for at least 3 API surfaces

Mature application is complete when:

- Error classification improved
- Test coverage added
- Documentation updated

---

## 9. Future Extensions (Non-Binding)

The design should allow future evolution into:

- CI regression testing tool
- Endpoint certification utility
- API compatibility diff tool
- Integration validation system for agent frameworks

---

## 10. Document Status

This specification is a living document and will be refined after:

- Architecture review
- Proof-of-concept implementation
- Prototype feedback cycle

