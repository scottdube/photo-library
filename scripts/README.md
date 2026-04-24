# Scripts

CLI utilities and one-off jobs. Anticipated contents (not yet written):

- `index.py` — full library indexing run
- `index_incremental.py` — process new photos only (called by launchd)
- `query.py` — CLI semantic query (for testing without web UI)
- `setup_db.py` — initialize SQLite schema with sqlite-vec extension

Run with `uv run python scripts/<name>.py`.
