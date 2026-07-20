---
name: lightweight
description: >
  Audit a codebase to keep a product lightweight — small footprint, fast hot paths,
  no dead weight — without cutting quality. Use when the user wants a performance
  audit or to slim a tool/app/product (bloat, latency, memory, bundle size). Use
  when another skill needs a dedicated performance pass.
disable-model-invocation: true
---

# Lightweight

Make the product **lightweight**: low resource cost and fast **hot paths**, with every gram of weight *earning* its keep. **Load-bearing** quality stays — correctness, tests, types, security, and honest error handling are not fat to trim.

You are an auditor first. Implement only when the user asks; otherwise hand them ranked, evidenced findings they can act on.

## Hard rules

1. **Measure before ranking.** Guessing which path is slow is not a finding. Capture a **baseline** (or state why one is unavailable) before you prioritize work.
2. **Hot path first.** Cost on a rare admin route is noise; cost on startup, the primary request, or every render multiplies. Spend legwork where traffic lives.
3. **Load-bearing stays.** Never recommend dropping tests, type safety, authz, input validation, or user-visible error handling to win weight. If a cut would weaken quality, reject it or reframe it as a design change that preserves the floor.
4. **Evidence or out.** Every finding needs `file:line` (or a measured metric with the command that produced it). Vibes are not findings.
5. **Pareto, not polish.** Prefer algorithmic and architectural wins (N+1, wrong complexity, unbounded payloads, dependency bloat on the critical path) over micro-optimizations that save nanoseconds and cost readability.

## Steps

### 1. Lock scope and budgets

Pin what "lightweight" means for *this* product before reading code:

1. **Surface** — CLI, library, backend service, frontend/web, mobile, monorepo slice, or "whole product."
2. **Budget** — at least one checkable limit the user cares about. Examples: cold start / CLI time-to-first-output, p95 latency, RSS / peak memory, install size, JS bundle KB, LCP/INP, dependency count on the critical path. If the user has none, propose 1–3 from the surface and get agreement (or record defaults and say so).
3. **In / out of scope** — packages, routes, binaries, or paths to ignore.
4. **Mode** — `audit` (default: report only) or `fix` (audit, then implement agreed findings).

Completion: you can state in one short block: surface, budgets (or explicit defaults), scope boundaries, and mode. If budget is still undefined after a reasonable recon of existing docs/CI, pick surface-default budgets, name them, and proceed — do not stall.

### 2. Recon

Map the product enough to know where weight lives:

- Entry points (binaries, HTTP routes, UI roots, library public API).
- Build / test / start / package commands from README and manifests.
- Dependency graph shape: heavy runtime deps, optional vs required, tree-shaking / code-split setup.
- Existing perf gates (bundle analyzer scripts, Lighthouse CI, benchmarks, `hyperfine`, k6, etc.).
- Intent docs if present (`CONTEXT.md`, ADRs, `PRODUCT.md`) — decided tradeoffs are not findings.

Completion: a short recon note naming entry points, how to run/measure, the critical path you will treat as hot, and any recorded perf tradeoffs.

### 3. Baseline

**Measure** the budgets from step 1 with the cheapest honest method:

| Surface | Prefer |
|---|---|
| CLI / lib | timed invocation on a fixed fixture; `hyperfine` if present; RSS via `/usr/bin/time -v` or equivalent |
| Backend | one representative request timed end-to-end; query counts; heap if memory is in budget |
| Frontend / web | bundle size from production build; Lighthouse/PSI or DevTools when a URL/report exists; waterfall of first paint path |
| Any | existing benchmark suite if it covers the hot path |

If you cannot measure (no runnable build, no fixture, no URL), say so, switch to **static-only** mode, and label every finding `unmeasured — potential impact`. Do not invent numbers.

Completion: each budget has either (a) a baseline number + the exact command/output that produced it, or (b) an explicit "unmeasured" tag with the blocker. Paste commands and results you actually ran.

### 4. Audit

Hunt **dead weight** and hot-path cost. Read the pattern catalog for every surface in scope — [references/surfaces.md](references/surfaces.md) — and apply every section that matches.

For each candidate finding:

- Cite evidence (`file:line` and/or measured delta).
- Name the budget it threatens.
- Estimate impact (H/M/L) and effort (S/M/L).
- Note whether the fix is load-bearing-safe (preserves quality floor).

Reject micro-opts that do not move a budget. Reject "remove X" when X is load-bearing unless a quality-preserving alternative is part of the finding.

Completion: every applicable section of `surfaces.md` for in-scope surfaces has been considered; each kept finding has evidence, budget link, impact, effort, and a load-bearing-safe fix sketch. Sections considered with no hits are fine — do not pad.

### 5. Rank and gate

Order findings by **leverage** (impact ÷ effort), then by confidence. Apply the quality gate:

- Drop or rewrite any recommendation that trades away load-bearing quality for weight.
- Promote fixes that *raise* quality while cutting weight (delete dead code, shrink public surface, replace a heavy dep with a small one that still covers the need).
- Mark `by-design` anything the recon ADRs already accepted; only report implementation that exceeds the documented tradeoff.

Completion: a ranked table with no ungated items; every row would move a named budget without lowering the quality floor.

### 6. Report (and optionally fix)

Deliver:

1. **Budgets + baselines** (or unmeasured status).
2. **Findings table**: `# | Finding | Budget | Impact | Effort | Evidence`.
3. **Top 3 moves** — the Pareto cut: what to do first and the expected budget movement (measured or reasoned).
4. **Out of scope / rejected** — one line each so they are not re-litigated next run.
5. **What was not audited** — honest gaps (packages skipped, no prod build, etc.).

In `audit` mode, stop here unless the user selects findings to implement.

In `fix` mode: implement only selected (or top-ranked if they said "just fix it") findings; re-run the baseline commands from step 3; report before/after for each touched budget. Hand performance *regressions* that need a tight repro loop to `diagnosing-bugs`.

Completion: user can act without re-reading the codebase — evidence, ranking, and next actions are all in the report. Fix mode also includes before/after measurements for every changed budget.

## Relationship to other skills

- **`improve`** — broad multi-category advisor; use this skill when performance/weight is the whole job, or when improve needs a deeper perf pass.
- **`diagnosing-bugs`** — a known slowdown with a symptom; build a tight red loop there, then return here for structural weight once the regression is fixed.
- **Web-only Core Web Vitals deep dives** — if a Lighthouse/CrUX/trace workflow already exists in the host (`/webperf` or similar), use it for browser metrics and fold its scorecard into this skill's budgets; still run the code-level audit here for deps and architecture.
