# vLLM LTS Ops Dashboard

End-to-end workflow for building, editing, screenshotting, and incident-analyzing Grafana dashboards whose datasource is Huawei Cloud LTS over vLLM/ModelArts engine logs. Chinese version: [README_zh.md](README_zh.md).

## What it covers

- LTS query semantics over pre-aggregated metric lines (Engine ticks, fleet throughput, KV cache / prompt / generation throughput).
- How to aggregate per-engine metrics correctly, including the sum-of-per-engine-max form for fleet totals.
- Dashboard build/edit workflows over Grafana's database.
- PNG capture of dashboards whose rows are collapsed by default (render API and manual capture).
- Preemption and slow-TPOT incident analysis from metric-line signatures.

No credentials or environment identifiers are stored in this repo.

## Requirements

- Grafana with the Huawei LTS datasource plugin (`hw-hws-lts-grafana-datasource-plugin`).
- vLLM/ModelArts engine logs streaming into LTS as Engine metric lines.

## Related skills

- `grafana-dashboards` — general Grafana setup, auth, and renderer.
- `grafana-dashboard-dev` — the portable engineering workflow this LTS layer extends.

## License

MIT
