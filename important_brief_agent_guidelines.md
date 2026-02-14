# IMPORTANT_BRIEF_AGENT_GUIDELINES.md

Minimal operating rules for Codex or other automated agents in this repository.

---

## Core Authority

- SPEC.md is authoritative.
- Do not change requirements silently.
- Stop at review gates.

---

## Required Workflow

Before work:
- Read SPEC.md
- Read PROJECT_STATUS.md

After meaningful work:
- Update PROJECT_STATUS.md
- List changes made
- List open questions
- Recommend next step

---

## Architecture Rules

- HTTP logic → src/profiler/http_client.py
- Data models → src/profiler/models.py
- Probes → src/profiler/probes/
- CLI → src/profiler/main.py
- No mixing probe logic with report rendering

---

## Engineering Standards

- Python 3.13+
- Full type hints
- mypy strict
- ruff clean
- No global mutable state
- Prefer dataclasses

---

## Error Handling

- Distinguish: 404 vs 400 vs auth vs network
- Record status codes in results
- Do not suppress unexpected errors

---

## Design Philosophy

- Prefer clarity over cleverness
- Avoid unnecessary abstraction
- Minimize token usage during probing
- Keep outputs deterministic

---

End of IMPORTANT_BRIEF_AGENT_GUIDELINES.md

