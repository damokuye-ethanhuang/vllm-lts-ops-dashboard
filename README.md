# vLLM O&M Dashboard Dev / vLLM 运维看板开发

End-to-end workflow for building, editing, screenshotting, and incident-analyzing Grafana dashboards whose datasource is Huawei Cloud LTS over vLLM/ModelArts engine logs. Chinese version: [README_zh.md](README_zh.md).

## Contents

- [Capabilities](#capabilities)
- [Requirements](#requirements)
- [Known Pitfalls](#known-pitfalls)
- [Related Skills](#related-skills)
- [Documentation](#documentation)
- [License](#license)

## Capabilities

| Capability | What it gives you |
|---|---|
| LTS query semantics | How to query pre-aggregated metric lines (Engine ticks, fleet throughput, KV cache / prompt / generation throughput) without inflating results 10–200×. |
| Per-engine aggregation | Correct fleet-total forms, including the sum-of-per-engine-max pattern and its ~1% verification. |
| Dashboard build & edit | Scripted edit workflow over Grafana's database: re-GET, version bump, rewrite assertions. |
| PNG capture | Screenshot paths for dashboards whose rows are collapsed by default — render API and manual capture both covered. |
| Incident analysis | Preemption and slow-TPOT triage from metric-line signatures, with explicit rules for what the log stream cannot prove. |

## Requirements

| Component | Requirement |
|---|---|
| Grafana | With the Huawei LTS datasource plugin (`hw-hws-lts-grafana-datasource-plugin`) |
| Log stream | vLLM/ModelArts engine logs flowing into LTS as Engine metric lines |

## Known Pitfalls

A few things worth knowing before you query LTS; the full list is in `SKILL.md`:

- Metric lines are ticks, not samples — `sum()` over a gauge/rate column inflates 10–200×; use `max()` for bounded per-engine gauges.
- Aggregate-of-aggregate in one SELECT fails with `LTS.2801`; flatten through a grouped subquery.
- The LTS front door throttles per AK at low double-digit req/s — probe loops need sleep between queries or panels come back empty.

## Related Skills

- `grafana-dashboards` — general Grafana setup, auth, and renderer.
- `grafana-dashboard-dev` — the portable engineering workflow this LTS layer extends.

## Documentation

| Document | Purpose |
|---|---|
| [SKILL.md](SKILL.md) | Full working procedure: query semantics, dashboard build/edit, capture, incident analysis, verification checklist |
| [README_zh.md](README_zh.md) | Chinese version of this README |

## License

MIT
