# photo-library

Local semantic image search over a personal Apple Photos library, served as
a web application over an existing WireGuard VPN. Built around CLIP-family
embeddings + SQLite vector search, running on a Mac Mini M4.

## Why

Apple Photos search fails to retrieve photos by subject matter for specific
hobby/reference vocabulary. Test results on a 22k-photo library:

- "CNC": 29 results, mostly novelty photos from week one of owning the machine
- "Tormach": 74 results, mostly noise (screenshots, t-shirts, filename matches)
- "Fixture plate": 0 results despite many existing in library

Manual keyword/caption discipline isn't a reliable solution. This project
builds a parallel semantic-search index, accessible from iPhone away from home
via existing VPN, with deep-link back into Photos.app for sharing.

See [docs/project-charter.md](docs/project-charter.md) for full scope, phases,
architecture, and validation strategy.

## Status

Phase 0 (CLIP validation via `rclip`) complete and greenlit — ~80% top-3
relevance across 8 representative queries on a 17,866-photo index,
including shop content (CNC, lathe, fixture plate) that Photos.app cannot
find. Phase 1 (sqlite-vec index, osxphotos pipeline, FastAPI service,
launchd persistence) not yet started. External SSD (Samsung 990 EVO Plus
4TB + OWC Express 1M2) on hand as of 2026-04-27 for the Photos library
migration that precedes full-library production indexing. See
[docs/session-log/session-2026-04-24.md](docs/session-log/session-2026-04-24.md).

## Stack

- Mac Mini M4 (LRD), 192.168.50.10
- Python 3.12, managed via `uv`
- CLIP-family embedding model (selection deferred to Phase 0)
- SQLite + sqlite-vec for vector storage
- FastAPI for query service
- Persistent service via launchd (matches existing Ollama/Open WebUI pattern)
- Access via WireGuard VPN through UDM Pro

## Repo Layout

```
docs/
  project-charter.md    Full project charter
  decisions/            Architecture decision records
  session-log/          Per-session notes
scripts/                CLI utilities, one-off jobs
src/photolib/           Application code
tests/                  Pytest suite
```

## Local Setup (placeholder)

To be filled in once Phase 1 begins. Anticipated:

```bash
uv sync                          # install deps from pyproject.toml
cp .env.example .env             # configure paths
uv run python scripts/index.py   # build initial index
uv run uvicorn photolib.api:app  # start query service
```

## Related

- `scottdube/network-docs` — overall LRD/SLN/UPL infrastructure documentation
- ADR-004 in network-docs — Mac Mini AI stack (Ollama, Open WebUI)
- ADR-001 in network-docs — LRD-Servers VLAN where Mac Mini lives
