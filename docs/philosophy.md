# Switchboard Philosophy

The principles behind the [architecture](./architecture.md). When a decision is
ambiguous, these are the tiebreakers.

## Boring tech, few moving parts

We optimize for someone else being able to run this with one command and minimal
decisions. Every extra required service is a reason someone bounces off the README.
Postgres is allowed to be a dependency; Redis, Kafka, a separate queue, an
orchestrator, and dbt are not. When a simpler tool covers our scale, we use it —
Postgres is our queue, cron is our scheduler, views are our rollups.

## The data is the user's

The hook sends transcripts to storage the user owns; Switchboard analyzes whatever
lands there. We never own the ingestion edge. This is a trust win for self-hosters
and keeps our release cycle decoupled from theirs.

## The OpenTranscripts spec is sacred

The canonical standardized format — the **OpenTranscripts spec** — is the contract
every other part of the system, and every third-party tool, depends on. It is
versioned, published as a JSON Schema, and validated on write. Raw transcripts stay
a faithful, uninterpreted capture; the spec is where we declare canonical fields
(total tokens, cost, timing). Getting it right is cheap today and ruinously
expensive to fix in a year, so it gets disproportionate care.

## Append-only, never overwrite

Analysis results are versioned and additive. Re-running is idempotent; changing a
prompt or producer version produces new rows beside the old ones. Provenance
(source, producer, model, prompt hash) rides inline on each segment and outcome.
Storage is cheap; the audit trail and free A/B comparison of prompt versions are
worth far more than the bytes. We never destroy the evidence of how a number was
produced.

## Configuration over redeployment

Sources, segmenters/scorers, prompts, models, and feature flags are *data and env
vars*, not hardcoded constants. A self-hoster — or we ourselves — should be able to
change behavior without editing code and shipping a container. This is the single
most important move for the self-host experience.

## Design for the next version, pay for this one

We build the schema and seams that v2/v3 need when they're nearly free today
(segments as a shared write target, service tokens, the extension seam for
recommendations), but we do not *build* v2/v3 early. The MVP ships with no LLM
calls at all — deterministic parsers first, real transcripts in hand before we
invest in LLM analysis.

## Dogfood through the same door as everyone else

Our production instance runs the same image, the same compose shape, and adopts
releases the same way an external self-hoster would. Internal-only code lives in a
private infra repo as configuration, or upstream behind a flag — never as a
divergent fork. If we feel a papercut, so will our users, and we fix it in public.

## Effect where it earns its keep, plain TS at the edges

The pipeline is a graph of fallible, heterogeneous steps with retries, concurrency,
and (eventually) LLM orchestration — Effect's sweet spot. We use it for parsers,
workers, schemas, and the API, and keep React components and Next.js pages as plain
TypeScript. The engine is typed and composable; the edges stay boring.

## Self-host experience is a feature

`docker compose up` should produce a working app in a minute. Migrations run on
boot. Defaults are safe and refuse to run insecurely in production. There's a path
to try it before wiring up real data. Good docs and a clean upgrade story are part
of the product, not an afterthought.
