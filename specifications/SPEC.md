# SPEC.md

This is the high-level specification for the LLM Endpoint Capability Profiler.

The application shall:

- Accept an API base URL and authentication credentials.
- Discover supported API surfaces (chat completions, responses, embeddings, etc.).
- Probe feature capabilities (tool calling, streaming, JSON mode, etc.).
- Enumerate available models.
- Generate a Markdown capability report.
- Optionally generate machine-readable JSON output.

This document will be refined during Phase 1 of development.
