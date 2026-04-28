# photo-library — Working Instructions

Working instructions for Claude (and future-Scott) on this repo. This
file holds AI working preferences and pointers to canonical project
state. Project facts (status, stack, layout, conventions) live in their
canonical home files; this file points to them rather than duplicating,
to prevent drift.

## Where to look for project state

- **What this project is and why:** `README.md`, `docs/project-charter.md`
- **Current phase / status:** `README.md` Status section + most recent
  file under `docs/session-log/`
- **Stack and dependencies:** `pyproject.toml` (authoritative),
  `README.md` Stack section (prose summary)
- **Repo layout:** `README.md`
- **Non-goals and success criteria:** `docs/project-charter.md`
- **How to run things:** `README.md` Local Setup section
- **Conventions:** `docs/decisions/README.md` (ADRs),
  `docs/session-log/README.md` (session logs), `pyproject.toml` (Python
  style and test config), `.gitignore` (never-commit list)
- **Configuration keys:** `.env.example`
- **Anticipated scripts (planned, not yet written):** `scripts/README.md`

Two AI-instructional conventions that don't have a canonical home
elsewhere yet, kept here for now:

- **Session logs** follow the structural template of
  `docs/session-log/session-2026-04-24.md`: Summary, Setup, Methodology,
  Results, Key findings, Known issues, Decisions, Next steps, Time
  invested, Hardware/environment, Related artifacts.
- **Decisions inside session logs are tagged `[DECISION]`** for
  greppability. Promote to a numbered ADR under `docs/decisions/` when
  the decision is architectural and durable.

## Working style preferences (Scott)

- Direct, precise, no fluff. Cite the source file/line when stating a
  project fact.
- Flag when an answer is inference rather than fact.
- Don't invent UI steps, function names, or flags. If unsure, grep first
  or say so.
- Respect stated constraints (paths, hardware, model choice deferral).
- Anticipate downstream issues (tolerances, model fit, software quirks)
  rather than answering only the literal question.
- **Code blocks must contain only executable code — no inline comments,
  no `#`-prefixed annotations, no leading/trailing prose lines inside the
  fence.** Scott copy-pastes code blocks directly into runtime contexts
  where `#` characters cause errors. Put all explanation, section
  headers, and step labels in prose *outside* the code block. If a
  multi-step procedure needs labels, use multiple separate code blocks
  with prose between them rather than annotating inline.

## Related external context

- `scottdube/network-docs` — overall LRD/SLN/UPL infrastructure docs.
- ADR-001 (network-docs) — LRD-Servers VLAN where the Mac Mini lives.
- ADR-004 (network-docs) — Mac Mini AI stack (Ollama, Open WebUI), the
  pattern this project's launchd service should mirror.
