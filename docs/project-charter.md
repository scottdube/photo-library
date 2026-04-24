# Photo Library Semantic Search — Project Charter

**Status:** Planning
**Created:** 2026-04-24
**Owner:** Scott

## Problem Statement

Photos.app search on iOS/macOS fails to reliably retrieve photos by subject
matter. Specific failures observed on a 22,084-photo library (LRD Mac Mini,
2026-04-24):

- "CNC": 29 results, mostly from first week of owning the machine (novelty
  labeling decays over time)
- "Tormach": 74 results, mostly noise (screenshots, t-shirts with word,
  stray keyword matches, not semantic)
- "Fixture plate": 0 results despite many existing in library

Apple's 722-label closed taxonomy + filename/keyword matching is inadequate
for recalling:

1. Specific lab/shop projects (CNC, lathe, electronics builds)
2. Grandchildren moments by activity or context
3. Specific named persons (partially works; 13,051 unknown face clusters)
4. Informational reference photos (construction blocking, wiring,
   pre-drywall documentation)

Manual caption/keyword habit is not reliable enough to close the gap.

## Goal

Build a local semantic image search system that retrieves photos by
natural-language description, accessible from iPhone away from home network.

Primary use case: "At a BBQ, I want to show someone a photo of my fixture
plate on the Tormach" — from query to photo-in-Photos.app in <20 seconds.

## Non-Goals

- Replacing Photos.app as the primary photo store. Apple Photos + iCloud
  continues as source of truth.
- Writing AI-generated keywords back into Photos.app (architectural
  dead-end; locks in model choice, clutters keyword list).
- Building an iOS app. Web UI over VPN is sufficient for scope.
- Replacing Apple's face recognition. Manual curation of _UNKNOWN_ faces in
  Photos.app is the lighter-weight fix where needed.
- Running vision-model captioning on entire library. CLIP embeddings are
  better + faster for semantic retrieval than LLM-generated captions.

## Architecture

Local CLIP-family embedding + vector similarity search, served as a web
application behind the existing WireGuard VPN.

```
+-- iPhone (anywhere) ----------------+
|  WireGuard on-demand -> UDM Pro     |
|  Browser -> photofind.home.arpa     |
|  Tap result -> Photos.app deep link |
+--------------+----------------------+
               | (HTTPS over VPN)
+--------------v----------------------+
|  Mac Mini M4 @ 192.168.50.10        |
|  +-------------------------------+  |
|  | FastAPI query service :8081   |  |
|  |   ↓                           |  |
|  | CLIP model (mlx-clip)         |  |
|  |   ↓                           |  |
|  | SQLite + sqlite-vec index     |  |
|  |   ↓                           |  |
|  | Photos library (via osxphotos)|  |
|  +-------------------------------+  |
+-------------------------------------+
```

**Key decisions:**

- **Embedding model:** CLIP or SigLIP-family, running via MLX on Apple Silicon
  for native performance. Specific model selection deferred to pre-build
  evaluation.
- **Vector store:** SQLite with `sqlite-vec` extension. Single file, no
  additional service, fits existing tooling style.
- **Index update:** nightly launchd job runs incremental indexing for new
  photos.
- **Access:** web UI on Mac Mini, served over existing WireGuard VPN.
  On-demand VPN activation on iPhone removes friction.
- **Deep-linking:** tap result -> iOS Photos.app deep link by UUID. Fallback:
  full-resolution image in web UI, save-to-camera-roll.

## Validation Strategy

Before committing full build (20-25 hrs), validate core hypothesis:

**Pre-flight 1: CLIP quality on representative subset**

- Install `rclip` (or equivalent off-the-shelf CLIP search tool) via uv on
  Mac Mini
- Point at ~500 photo subset from lab/shop/construction categories
- Run 10 representative queries ("fixture plate on mill", "CNC enclosure
  wiring", "bathroom blocking before drywall", etc.)
- Evaluate: do results contain what you were looking for in top 10?
- **Go/no-go criterion:** if CLIP misses obvious matches, evaluate SigLIP-2 or
  abandon project

**Pre-flight 2: iOS Photos.app deep-link verification**

- Confirm current iOS supports opening specific photo by UUID via URL scheme
  (historically has worked via `photos-redirect://` or similar, but Apple
  periodically revises)
- If not supported, adjust UX to full-res image display with share-sheet
  fallback

## Phased Buildout

**Phase 0 — Pre-flight** (~2 hrs)

- Install rclip, test on subset, evaluate quality
- Verify iOS deep-link mechanism
- Decide: proceed, alternate approach, or abandon

**Phase 1 — Indexing pipeline** (~8-10 hrs)

- Export photo metadata + UUIDs + file paths via osxphotos -> SQLite
- Generate CLIP embeddings for all ~22k photos (one-time; ~30-60 min
  compute on M4)
- Store embeddings in sqlite-vec index alongside metadata
- Incremental re-indexing script for new photos
- launchd job scheduled nightly

**Phase 2 — Query service** (~6-8 hrs)

- FastAPI application: text query -> CLIP text embedding -> vector similarity
  search -> ranked results
- Simple web UI: search input, thumbnail grid, click-to-open
- Thumbnail generation/serving (may use Photos.app native thumbnails via
  osxphotos)
- launchd service for persistence
- Installed via `uv tool install` pattern consistent with Ollama/Open WebUI

**Phase 3 — Mobile access** (~2-4 hrs)

- DNS: `photofind.home.arpa` (or similar) on UDM Pro -> Mac Mini
- HTTPS certificate (local CA or Let's Encrypt via Cloudflare DNS challenge)
- iPhone WireGuard on-demand activation rule for the FQDN
- Mobile-responsive UI: larger touch targets, fast-loading thumbnails
- Deep-link integration or fallback flow
- Bookmark on iPhone home screen

**Phase 4 — HA integration** (~4 hrs, optional)

- HA REST command wrapping the query service
- Custom sensor for "photo count by query"
- Voice intent: "do I have photos of X" -> HA -> query service -> notification
  with count and representative thumbnail

**Estimated total:** 20-25 hours, spread over several weekends.

## Success Criteria

1. Given a natural-language query describing photo content, top-10 results
   contain the target photo >=80% of the time (for queries where target
   exists in library).
2. End-to-end mobile workflow (phone lock -> query -> photo open in Photos)
   takes <=20 seconds when VPN connected.
3. System runs unattended on Mac Mini, nightly index updates succeed without
   intervention.
4. Zero maintenance burden on Photos.app source-of-truth behavior (no
   keyword pollution, no iCloud sync interference).

## Open Questions (to resolve during Phase 0)

- Best CLIP-family model for Apple Silicon in 2026 (mlx-clip, SigLIP-2 via
  sentence-transformers, other)
- iOS Photos.app deep-link URL scheme current status
- Thumbnail serving strategy (regenerate vs. use Photos.app's cached
  thumbnails)
- Photos library path stability (does it move on macOS updates?)

## Related Documents

- `scottdube/network-docs` repository — overall LRD/SLN/UPL infrastructure
  documentation
- ADR-004 in network-docs — Mac Mini AI stack (Ollama, Open WebUI)
- ADR-001 in network-docs — LRD-Servers VLAN where Mac Mini lives
- network-docs `docs/changelog.md` 2026-04-24 — Photos sync + osxphotos
  baseline session
