# placeboagent

A fake [AG-UI](https://docs.ag-ui.com) agent backend for testing.
Scriptable, deterministic, offline.

> **Status: pre-release.** 0.1.0 is in active development.
> The full roadmap lives in [docs/PLAN.md](docs/PLAN.md).

Part of the placebo family of test doubles:
[placebopay](https://placebopay.dev) (fake payment gateway) ·
[placeborag](https://github.com/elaz48/placeborag) (fake RAG stack) ·
**placeboagent** (fake agent backend).

## Why

If you build AG-UI frontends, clients, or emitters, your test target today
is a real agent backend: slow, nondeterministic, token-metered, and
incapable of producing the failures that actually break UIs in production.
Dropped SSE connections mid-message. Malformed events. Lifecycle events out
of order. A tool call that never finishes.

placeboagent speaks the AG-UI protocol with no LLM behind it. Same
scenario, same byte stream, every time. All event types come from the
official [`ag-ui-protocol`](https://pypi.org/project/ag-ui-protocol/)
package, so conformance tracks the spec.

## Planned for 0.1.0

- **Scriptable fake agent.** An ASGI app exposing one AG-UI SSE endpoint.
  Define a run as an ordered script of events with timing control: instant,
  fixed delay, or realistic token pacing.
- **Canned scenarios.** Plain text reply, streamed tool call, state
  snapshot + deltas, human-in-the-loop interrupt, run error.
- **Record & replay.** Capture the event stream of a real AG-UI backend to
  JSONL, then serve it back verbatim or at adjusted speed. Recorded
  sessions become fixtures you commit and replay in CI.
- **Failure injection.** Drop the connection after event N, stall
  mid-stream, emit malformed or unknown events, violate lifecycle
  ordering, duplicate events.
- **pytest plugin.** A `placebo_agent` fixture plus assertion helpers
  (`assert_valid_run`) for testing your own AG-UI emitters.

Runs standalone (`placeboagent serve`) for frontend developers who never
touch Python, or mounts into any FastAPI/Starlette app.

## Target API preview

Subject to change until 0.1.0 ships. This is the design being built, not
a released interface.

```python
from placeboagent import PlaceboAgent, scenarios

agent = PlaceboAgent(scenario=scenarios.streamed_tool_call())
app.mount("/agent", agent)   # any FastAPI/Starlette app
```

```bash
placeboagent serve --scenario text-reply --port 8199
```

## Non-goals

No LLM calls, ever (that is the point). No hosted service, no UI, no
per-framework integrations: the protocol is the interface.

## License

MIT
