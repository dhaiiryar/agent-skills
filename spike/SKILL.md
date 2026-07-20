---
name: spike
description: Research a spike topic and write up the results. Use when the user wants a spike, to investigate a technical topic, or to research and report findings before building.
disable-model-invocation: true
---

# Spike

Research a **spike** topic and write the **results**. The product is a findings document, not a feature.

## 1. Take the topic

State the spike topic in one line from the user's request. If it is ambiguous, ask once; otherwise proceed.

**Done when:** the topic is written and visible.

## 2. Research

Investigate the topic against what can settle it:

1. Code, configs, tests, and docs in the repo
2. Small throwaway experiments when reading cannot answer
3. Primary sources outside the repo — official docs, specs, first-party APIs

Follow every claim back to its source. Prefer evidence over opinion.

**Done when:** enough evidence exists to write results that stand alone — the reader should not need to re-run the investigation.

## 3. Write the results

Write a single Markdown results file covering:

- **Topic** — what was spiked
- **Findings** — what is true, with sources (paths, commands, links)
- **Options** — approaches considered, with trade-offs when more than one exists
- **Recommendation** — what to do next, and why
- **Open questions** — what remains unknown

Save it where the repo already keeps spike or research notes; if there is no convention, put it somewhere sensible (e.g. `docs/spikes/<topic>.md`) and say where.

**Done when:** the file exists, every section above is filled (open questions named, not omitted), and you tell the user the path.
