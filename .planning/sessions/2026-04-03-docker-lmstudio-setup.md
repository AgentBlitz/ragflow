# Session: Docker Deployment with LM Studio Integration

**Date:** 2026-04-03
**Branch:** main
**Commit:** (pending)

## Summary

Set up RAGFlow to run locally via Docker on macOS Apple Silicon (M4, 64GB RAM) with LM Studio as the LLM backend. Resolved a container crash caused by the v0.24.0 image being incompatible with the repo's current `entrypoint.sh` (which expects `ragflow.conf.python` nginx config not present in that image). Switched to the `nightly` image, enabled macOS optimizations, configured timezone, and used Playwright to log in and add LM Studio as a model provider with both a chat model (`qwen/qwen3-4b-2507`) and an embedding model (`nomic-ai/nomic-embed-text-v1.5-GGUF`), setting both as defaults. Updated CLAUDE.md with deployment notes. Docker stack was shut down at end of session.

## Changes

- `CLAUDE.md` — Added two new sections: "Local Docker Deployment (macOS Apple Silicon)" documenting the nightly image requirement, MACOS=1 flag, TZ setting, and login API quirk; and "LM Studio Integration" documenting host.docker.internal connectivity, provider setup, and embedding model requirement.
- `docker/.env` — Changed `RAGFLOW_IMAGE` from `v0.24.0` to `nightly`; set `TZ=Europe/London`; uncommented `MACOS=1`.

## Decisions & Rationale

- **Nightly image over v0.24.0** — The repo's `main` branch `entrypoint.sh` mounts into the container and references `ragflow.conf.python`, which only exists in builds newer than v0.24.0. The tagged image crashed in a restart loop. Nightly matches the current codebase.
- **LM Studio provider over OpenAI-API-Compatible** — RAGFlow has a dedicated LM-Studio provider which is cleaner than the generic OpenAI-compatible option.
- **nomic-embed-text-v1.5-GGUF for embeddings** — Small (~260MB), well-supported in LM Studio, good quality for local use.

## Remaining Work

- User needs to **download and load `nomic-ai/nomic-embed-text-v1.5-GGUF`** in LM Studio before document parsing will work
- Create a knowledge base, upload documents, and test the full RAG pipeline end-to-end
- Consider whether the `.env` changes should be gitignored or kept on a local branch (they contain local-specific settings)
- No ARM-native RAGFlow image exists yet — performance under x86 emulation is acceptable but OCR/parsing will be slower

## Resumption Context

Key files to review when picking this up:
- `docker/.env` — Local deployment config (image, timezone, macOS flag)
- `docker/docker-compose.yml` — Service definitions, port mappings, volume mounts
- `docker/service_conf.yaml.template` — Backend service config (LLM defaults can be pre-configured here)
- `docker/entrypoint.sh` — Startup script (source of the v0.24.0 crash — lines 184-199 nginx config selection)
- `CLAUDE.md` — Updated with deployment and LM Studio notes

Suggested opening prompt for next session:
> "RAGFlow is set up for Docker deployment with LM Studio. The stack is currently down (`docker compose up -d` from `docker/` to restart). I need to load the embedding model in LM Studio, then create a knowledge base and upload documents to test the RAG pipeline. See `.planning/sessions/2026-04-03-docker-lmstudio-setup.md` for full context."
