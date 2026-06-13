# Switchboard

Free, self-hostable open-source software that saves agent transcripts from Claude Code, Codex, and other coding agents for retroactive analysis. The analysis surfaces where an organization can swap out models and harnesses to optimize its token spend.

## Project Status

Switchboard is at an early, pre-implementation stage. The repository holds agent instructions (`AGENTS.md` / `CLAUDE.md`), a `README`, and the design docs under `docs/` — there is no application code, build system, or test suite yet. The sections below describe intended scope and conventions; expand them with concrete detail (stack, folder hierarchy, build/test/run commands) once the codebase grows, and keep this file aligned with the code.

## Repository Layout

```
.
├── AGENTS.md          # this file — agent instructions (mission, principles, conventions)
├── CLAUDE.md          # identical copy of AGENTS.md for Claude Code
├── README.md          # human-facing intro + pointer to docs/
└── docs/
    ├── architecture.md  # canonical architecture reference (diagrams + data model) — start here
    └── philosophy.md    # the "why" behind the architecture; tiebreakers for ambiguous calls
```

## Architecture at a Glance

The full reference is [docs/architecture.md](./docs/architecture.md); the load-bearing one-liners:

- **Four-stage pipeline:** `raw_transcripts` → `open_transcripts` → `segments` → `outcomes`.
- **One determinism boundary:** raw → OpenTranscript is a deterministic, versioned ETL; OpenTranscript → segments → outcomes are LLM-driven (and land in v2 — the MVP makes no LLM calls).
- **The OpenTranscripts spec is the contract:** one versioned, JSON-Schema-validated canonical format every other part and every third-party tool targets.
- **Sources are data, not code:** a source is an S3-compatible `storage_uri` plus an `adapter` that reads it and writes `raw_transcripts`. Amazon S3 ships built-in; other shapes get a custom adapter. `raw_transcripts` keeps the payload verbatim plus promoted sidecar columns (harness + version, model, actor, trigger, tokens, cost).
- **Append-only analysis:** segments/outcomes are version-tagged and idempotent; outcomes attach to segments, never transcripts; provenance rides inline (no separate analyzers tables).
- **Postgres is the only required dependency** — and the job queue (`FOR UPDATE SKIP LOCKED`): no Redis, Kafka, or external orchestrator.
- **One Docker image** (Next.js web + graphile-worker), migrations on boot, full-stack TypeScript with Effect in the engine.
- **Auth starts admin-only** (env-var password + scoped service tokens); SSO + role-based access are the planned trajectory, not the end state.

## What Switchboard Does

- **Captures** agent transcripts from multiple coding agents — Claude Code, Codex, and others.
- **Stores** those transcripts so they can be reviewed after the fact rather than only in the moment.
- **Analyzes** captured runs to reveal where a different model or harness would have produced comparable results at lower cost.
- **Optimizes token spend**: the analysis is the means; reducing an organization's model/harness costs is the end.
- Ships as **free, self-hostable OSS** — an organization runs it on its own infrastructure with no dependency on a hosted service.

## Core Principles

These are derived from the project's mission. Treat them as load-bearing constraints when proposing changes.

### Agent-agnostic
Switchboard ingests transcripts from many coding agents. Avoid coupling the core to any single agent's transcript format — normalize at the edges so capture, storage, and analysis stay agent-neutral. Adding support for a new agent should be an additive ingestion concern, not a core rewrite.

### Self-hostable first
Every feature must be runnable on a user's own infrastructure. Do not introduce hard dependencies on proprietary hosted services for core functionality. Hosted or managed offerings, if any, are additive — never a precondition for the OSS to work.

### Free and open source
The project is OSS. Keep licensing, dependencies, and contribution flow compatible with that commitment.

### Treat transcripts as sensitive data
Agent transcripts can contain source code, secrets, and proprietary context. Keep data local by default, avoid exfiltrating transcripts to third parties without explicit operator consent, and be conservative with logging and retention.

## Development

The repository has no build system, dependencies, or tests. Once the first implementation lands, document here:

- The language/runtime and how to install dependencies
- How to build, run, and self-host the service locally
- How to run the test suite (and which tests to run for targeted changes)

Until code exists, changes are documentation- or scaffolding-only.

## Contributing

Work on a feature branch off `main` and open a pull request for review — do not push directly to `main`. Refine this section as contribution conventions (CI, linting, commit/PR format) are established.
