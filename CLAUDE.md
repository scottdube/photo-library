# photo-library — Working Instructions

Working notes for Claude (and future-Scott) on this repo. Keep this file
short, factual, and current. When something here contradicts the code, fix
the code or fix this file — don't let drift accumulate.

## What this project is

Local semantic image search over Scott's Apple Photos library, served as a
web app over an existing WireGuard VPN. CLIP-family embeddings + SQLite
vector search, running on a Mac Mini M4 (LRD, 192.168.50.10). Source of
truth for photos remains Apple Photos / iCloud — this is a parallel index,
not a replacement.

Source: `README.md` lines 3–6, 14–18; `docs/project-charter.md` lines 32–48.

## Status (as of 2026-04-27)

- **Phase 0 — Pre-flight: complete and greenlit.** rclip + base OpenCLIP
  on 17,866 photos hit ~80% top-3 relevance across 8 representative
  queries. Shop content (CNC, lathe, fixture plate) that Photos.app can't
  find is retrievable. Source: `docs/session-log/session-2026-04-24.md`
  lines 5, 26–38, 60.
- **Phase 1 — Indexing pipeline (sqlite-vec, osxphotos, launchd):
  not started.** Source: `docs/project-charter.md` lines 121–128;
  `README.md` line 25.
- **External SSD on hand as of 2026-04-27.** Samsung 990 EVO Plus 4TB +
  OWC Express 1M2 (~$749, ordered 2026-04-24) has arrived. Photos library
  migration to external SSD is the next hardware step before full-library
  production indexing. Original order rationale:
  `docs/session-log/session-2026-04-24.md` line 68.
- **Outstanding:** ~1,452 iCloud-only photos return `None` filepath via
  osxphotos; needs investigation before production index. Source:
  `docs/session-log/session-2026-04-24.md` lines 53–54, 70.

## Stack

- Python 3.12+, managed via `uv` (`pyproject.toml` lines 6, 19;
  `README.md` lines 30, 53–58).
- CLIP-family embedding model — **specific model deferred.** Phase 0 used
  default OpenCLIP via rclip 2.1.6; charter anticipates mlx-clip on Apple
  Silicon. Source: `docs/project-charter.md` lines 78–80;
  `docs/session-log/session-2026-04-24.md` lines 9–10.
- SQLite + `sqlite-vec` extension for vector storage (`README.md` line 33;
  `docs/project-charter.md` lines 81–82).
- FastAPI + uvicorn for the query service (`README.md` line 34;
  `pyproject.toml` line 17 anticipated).
- Persistence via launchd, matching the existing Ollama / Open WebUI
  pattern on the same Mac Mini (`README.md` line 35;
  `docs/project-charter.md` lines 138).
- Access via WireGuard VPN through UDM Pro (`README.md` line 36;
  `docs/project-charter.md` lines 56–60).

Anticipated dependencies (not yet pinned): `osxphotos`, `mlx-clip`,
`sqlite-vec`, `fastapi`, `uvicorn`, `python-dotenv`. Source: `pyproject.toml`
lines 11–18.

Dev tooling pinned now: `pytest>=8.0`, `pytest-asyncio>=0.23`, `ruff>=0.4`,
`mypy>=1.10`. Source: `pyproject.toml` lines 22–27.

## Repo layout

```
docs/
  project-charter.md     Charter — scope, phases, success criteria
  decisions/             ADRs — NNN-title.md, network-docs convention
  session-log/           session-YYYY-MM-DD.md notes
scripts/                 CLI utilities (index.py, query.py, etc. — TBD)
src/photolib/            Application code (currently just __init__.py)
tests/                   pytest suite (empty until Phase 1)
```

Source: `README.md` lines 39–47; `docs/decisions/README.md`;
`docs/session-log/README.md`; `scripts/README.md`; `tests/README.md`.

## Conventions

**ADRs.** `docs/decisions/NNN-title.md`, zero-padded sequence. Format
mirrors network-docs: context, decision, alternatives considered,
consequences. Source: `docs/decisions/README.md`.

**Session logs.** `docs/session-log/session-YYYY-MM-DD.md`. Use the
2026-04-24 file as the structural template — Summary, Setup, Methodology,
Results, Key findings, Known issues, **\[DECISION\]**-tagged decisions, Next
steps, Time invested, Hardware/environment, Related artifacts. Source:
`docs/session-log/README.md`; `docs/session-log/session-2026-04-24.md`.

**Decisions inside session logs are tagged `[DECISION]`** so they're
greppable. Promote to a numbered ADR when the decision is architectural
and durable. Inference: based on the three `[DECISION]` lines in the
2026-04-24 log (lines 60, 62, 64) and the existence of a separate
`docs/decisions/` directory.

**Python style.** `ruff` line length 100, target `py312`, lint selects
`E,W,F,I,B,UP`. `mypy` strict mode, `python_version = 3.12`. Source:
`pyproject.toml` lines 36–48, 55–59.

**Tests.** `pytest`, `testpaths = ["tests"]`, `python_files = ["test_*.py"]`,
`asyncio_mode = "auto"`. Run via `uv run pytest`. Source:
`pyproject.toml` lines 50–53; `tests/README.md`.

**Config.** Copy `.env.example` to `.env`. Never commit `.env`.
Anticipated keys: `PHOTOS_LIBRARY_PATH`, `INDEX_DB_PATH`,
`THUMBNAIL_CACHE_DIR`, `CLIP_MODEL_NAME`, `QUERY_SERVICE_HOST`,
`QUERY_SERVICE_PORT`, `LOG_LEVEL`. Source: `.env.example` lines 1–24.

**Things never to commit.** `.env`, `*.db`, `*.sqlite*`, `data/`,
`embeddings/`, `thumbnails/`, `*.photoslibrary/`, model weight files
(`.gguf`, `.safetensors`, `.bin`, `.pt`, `.pth`), and full or partial
photo exports. Source: `.gitignore` lines 56–80.

## How to run things

Anticipated commands once Phase 1 lands (placeholder per `README.md`
lines 49–58):

```bash
uv sync                          # install deps from pyproject.toml
cp .env.example .env             # configure paths
uv run python scripts/index.py   # build initial index
uv run uvicorn photolib.api:app  # start query service
```

Anticipated scripts (not yet written): `scripts/index.py`,
`scripts/index_incremental.py`, `scripts/query.py`, `scripts/setup_db.py`.
Source: `scripts/README.md`.

## Non-goals (don't drift into these)

- Replacing Photos.app or writing AI-generated keywords back into it.
- Building a native iOS app (web UI over VPN is the scope).
- Replacing Apple's face recognition.
- Running vision-LLM captioning over the whole library (CLIP is faster +
  better for retrieval here).

Source: `docs/project-charter.md` lines 39–48.

## Success criteria (definition of done for the system)

1. Top-10 results contain the target photo ≥80% of the time when the
   target exists.
2. End-to-end mobile workflow (lock screen → query → photo open in
   Photos.app) ≤20 seconds with VPN connected.
3. Unattended nightly index updates on the Mac Mini.
4. Zero impact on Photos.app source-of-truth behavior (no keyword
   pollution, no iCloud sync interference).

Source: `docs/project-charter.md` lines 158–168.

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
- network-docs `docs/changelog.md` 2026-04-24 — Photos sync + osxphotos
  baseline.

Source: `README.md` lines 60–65; `docs/project-charter.md` lines 180–187.
