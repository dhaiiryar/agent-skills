# Surface pattern catalog

Load only the sections for surfaces in scope. Each bullet is a *hunt target*: confirm with evidence, then score against a named budget. Architectural and algorithmic wins first; micro-opts last and only if a budget moves.

---

## Shared (every surface)

- **Wrong complexity on the hot path** — nested scans, repeated `find`/`filter`/`includes` where a `Map`/`Set` belongs, O(n²) over data that grows with users.
- **Repeated expensive work** — same pure computation or fetch recomputed per call/render/request with stable inputs; missing memo at a clear function boundary (not cargo-cult `useMemo` everywhere).
- **Unbounded work** — lists/queries/files without pagination, limits, or streaming; "load everything then filter in memory."
- **Sync work that blocks the critical path** — crypto, image, parse, or network work on the main/request thread that belongs off-path (worker, queue, lazy init).
- **Eager init** — heavy modules imported at startup for features the default path never uses; fix with lazy import / deferred require that preserves API and tests.
- **Dead weight** — unreachable exports, feature flags fully rolled out still branching, copy-pasted utilities, unused files still in the build graph.
- **Logging/telemetry on the hot path** — sync disk I/O, huge structured payloads, or string-building always-on at info level in tight loops.
- **Resource leaks** — handles, connections, timers, subscriptions not released; inflate RSS over time and look like "we need more memory."

### Dependency hygiene (all surfaces)

- **Heavy dep for a thin need** — full lodash/moment/aws-sdk/etc. when one function or a 20-line helper would do; prefer small focused packages or inlined helpers *when* behavior and tests stay equivalent.
- **Duplicate capability** — two date libs, two HTTP clients, two schema validators on the runtime path.
- **Dev-only weight shipped to runtime** — test utils, storybook, mock servers reachable from production entry points.
- **Transitive bloat** — a small API pulling a large tree; check with the ecosystem's analyzer (`npm ls`, `pnpm why`, `cargo tree`, `go mod graph`) and cite the edge.
- **Install / publish size** — unnecessary files in the package (`files` field / `.npmignore`), fat native addons when optional, multi-arch binaries always downloaded.

Load-bearing check: replacing a dep is only a finding if the replacement preserves correctness, security posture, and testability — "delete the library" alone is not enough.

---

## CLI and local tools

Budgets often: time-to-first-output, cold start, RSS, install size.

- **Import graph at startup** — top-level imports of full frameworks, large parsers, or cloud SDKs before argv is parsed; defer until the subcommand needs them.
- **Help / `--version` cost** — printing help should not boot the whole app; version path should stay tiny.
- **Filesystem thrash** — recursive walks without ignore rules; re-reading the same config/file per invocation; missing caches for repo-wide scans.
- **Subprocess storms** — N child processes where one batch or a library call would do; shelling out when a stable API exists.
- **Binary / distribution weight** — bundling whole runtimes or assets unused by the default command; optional features not actually optional in the package.

---

## Libraries / SDKs

Budgets often: dependency count, publish size, import cost for the default entry.

- **Barrel files that defeat tree-shaking** — root `index` re-exports pulling the world; side-effectful module init.
- **Public surface sprawl** — forcing clients to take rare utilities as hard runtime deps; split optional entry points (`pkg/lite`, `pkg/node`, `pkg/browser`).
- **Peer vs direct deps** — bundling a framework the host already provides; duplicate React/etc. in the consumer app.
- **Type-only imports shipped as values** — unnecessary runtime retain when `import type` / `export type` would drop them.

---

## Backend services

Budgets often: p95 latency, queries per request, payload bytes, RSS, cold start (serverless).

- **N+1** — query/fetch per item in a loop; missing batching, join, or DataLoader-style coalesce. Cite the loop and the query site.
- **Over-fetch** — `SELECT *`, full aggregates where a projection/ID list suffices; GraphQL resolvers pulling graphs clients never ask for.
- **Missing or wrong indexes** — only when schema + query evidence both exist; flag "verify EXPLAIN" rather than inventing index advice from names alone.
- **Chatty fan-out** — sequential awaits that could be limited-parallel; waterfall of service calls on the critical path.
- **Cache gaps / stampedes** — stable data re-fetched every request; no TTL/ETag/conditional GET where the protocol allows; thundering herd with no singleflight.
- **Oversized responses** — unbounded lists, base64 blobs inline, debug fields left in prod serializers.
- **Pool / connection misuse** — new client per request; pool too small under concurrency (measure first).
- **Serverless cold path** — heavy init outside the handler still imported; large deployment packages.

---

## Frontend / web

Budgets often: JS KB (initial + route), LCP, INP, request waterfalls.

- **Bundle composition** — heavyweight deps for trivial UI (full chart/icon/date lib); confirm with analyzer (`rollup-plugin-visualizer`, `webpack-bundle-analyzer`, source-map-explorer).
- **Missing code-splitting** — rare routes/modals/admin in the main chunk; dynamic `import()` at route boundaries.
- **Render / data waterfalls** — request starts only after parent render; sequential client fetches that could be parallel or server-composed.
- **Client work that belongs on the server** — large data shaping, auth redirects, or HTML that could be SSR/RSC/static without harming UX.
- **Image / font weight** — unoptimized assets, missing dimensions (CLS), too many font weights/families on first paint.
- **Re-render thrash** — unstable props/context forcing large subtrees; lists without stable keys or virtualization when N is large *and* measured janky.
- **Hydration / shipping unused JS** — islands or partial hydration opportunities; polyfills for browsers you no longer support.

If a Lighthouse/CrUX/trace is available, attach metric deltas to findings; otherwise label potential impact.

---

## Build, CI, and developer weight

Only when the user cares about *product* lightness through ship speed, or CI cost is an explicit budget.

- Redundant install/build steps; missing dependency and build caches.
- Test suites serialized when independent; enormous fixtures loaded for unit scope.
- Production builds that disable minification, tree-shaking, or compression by misconfig.

Do not turn a product perf audit into a CI redesign unless asked.

---

## Finding sketch (use in the report)

```
Finding: <one line>
Budget: <named budget>
Evidence: <path:line> and/or <command → number>
Why it costs: <hot path / multiplicity>
Fix sketch: <load-bearing-safe change>
Impact / Effort: H|M|L / S|M|L
```
