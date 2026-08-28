# placeboagent — Project Plan

A fake AG-UI agent backend for testing. Scriptable, deterministic, offline.
Part of the "placebo" family: [placebopay](https://placebopay.dev) (fake payment
gateway), [placeborag](https://github.com/elaz48/placeborag) (fake RAG stack),
and now a fake agent backend speaking the AG-UI protocol.

**One-liner:** Develop and test AG-UI frontends and clients without an LLM,
an agent framework, or a network connection. Deterministic event streams,
recorded-session replay, and failure injection for the paths real agents
never exercise in demos.

## Why this exists

Everyone building on AG-UI (CopilotKit users, framework integrators, client
SDK authors) currently tests against a real agent backend: slow,
nondeterministic, costs tokens, and never produces the failure cases that
break UIs in production (dropped SSE connections mid-message, malformed
events, out-of-order lifecycles, interrupted tool calls). The AG-UI ecosystem
has demo apps (Dojo) and a debugging tutorial, but no dedicated test double.

## Scope

Timebox: 3 weeks part-time to 0.1.0. Kill criteria below.

### M0 — Claim + pipeline (day 1)

- Register `placeboagent` on PyPI (verified free 2026-08-28), create public
  GitHub repo under MIT.
- Release pipeline: same as placeborag (uv, ruff, pytest, tag-triggered
  PyPI publish via trusted publishing, no manual steps).
- Pin `ag-ui-protocol` as the only runtime dependency besides Starlette.
  All event types come from the official package, never hand-rolled, so
  conformance tracks the spec package version.

### M1 — Scriptable fake agent (week 1)

- `PlaceboAgent`: an ASGI app exposing one AG-UI endpoint (SSE transport
  first, matching the protocol default).
- Scenario API: define a run as an ordered script of AG-UI events with
  timing control (instant, fixed delay per event, realistic token pacing).
- Ships with canned scenarios: plain text reply, streamed tool call,
  state snapshot + deltas, human-in-the-loop interrupt, run error.
- Mountable into any FastAPI/Starlette app; also `placeboagent serve` CLI
  for frontend devs who never touch Python.
- Deterministic by default: same scenario, same byte stream, every time.

### M2 — Record, replay, break (week 2)

- Recorder: point it at a real AG-UI backend, capture the event stream to
  JSONL (one event per line, timestamped).
- Replayer: serve a recorded session back verbatim or at adjusted speed.
  Recorded sessions become fixtures: commit them, replay them in CI.
- Failure injection, the actual differentiator:
  - drop the connection after event N or after a byte offset
  - stall mid-stream (slow-loris) for reconnect/timeout testing
  - emit malformed JSON or unknown event types
  - violate lifecycle ordering (content before start, missing finish)
  - duplicate events for idempotency testing

### M3 — DX, conformance assertions, 0.1.0 (week 3)

- pytest plugin: `placebo_agent` fixture that spins up the server on a
  free port and tears it down.
- Assertion helpers for people testing their *own* AG-UI emitters:
  `assert_valid_run(events)` checks lifecycle ordering and pairing rules
  against the official types. Small, honest scope: not a certification
  suite, a smoke check.
- README with a 60-second quickstart, one GIF, framework-neutral wording.
- Publish 0.1.0. Then distribution, which is the real deliverable:
  - PR/issue to the ag-ui-protocol repo asking to be listed in ecosystem
    resources
  - post in the AG-UI Discord (#show-and-tell or equivalent)
  - one short launch write-up (dev.to or blog), reused as LinkedIn post

## Non-goals

- No LLM calls, ever. That is the point.
- No WebSocket transport in 0.1 (SSE is the default; add WS only if an
  actual user asks).
- No UI, no hosted service, no dashboard.
- No per-framework integrations (LangGraph, PydanticAI, ...): the protocol
  is the interface, staying below the frameworks is the moat.
- No certification/conformance authority claims. Assertion helpers are
  best-effort checks against the published types.

## Success metrics (review at +3 months, hard stop)

1. Listed in AG-UI official docs / ecosystem resources or a maintained
   awesome-list. This is the primary win condition (discovery channel).
2. Any acknowledgment from the CopilotKit/AG-UI maintainer circle
   (Discord reply, issue comment, retweet-equivalent).
3. At least 1-2 inbound conversations (recruiter, client, interview)
   that reference it.

Stars and downloads are recorded but are not success criteria. If none of
the three above happen by the review date: archive with a closing note in
the README, keep it on the CV, move on. Same exit as the PlaceboPay formula.

## Risks / decision log

- 2026-08-28: SSE chosen as the only 0.1 transport. Risk: protocol
  transport churn (precedent: MCP moved SSE to Streamable HTTP in
  March 2026). Mitigation: transport isolated behind one module;
  event model comes from `ag-ui-protocol` so spec bumps are a dependency
  update, not a rewrite.
- 2026-08-28: Sherlocking risk accepted. If CopilotKit ships official test
  utilities, pivot to contributing this upstream (offer donation to the
  ag-ui-protocol org) rather than competing. That outcome still satisfies
  success metric 2.
- 2026-08-28: `aguidoctor` (PyPI + GitHub both free) noted as a possible
  follow-up CLI in the langdoctor family (lint/conformance for AG-UI
  emitters). Explicitly out of scope for this project; do not scope-creep
  M3's assertion helpers toward it.
- Name decision: `placeboagent` over `agui-testkit` / `fakeagui` for brand
  continuity with placebopay/placeborag. Verified free on PyPI and GitHub
  on 2026-08-28.
