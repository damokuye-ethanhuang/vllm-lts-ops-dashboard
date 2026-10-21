# vLLM LTS Ops Dashboard / vLLM LTS 运维看板

End-to-end workflow for building, editing, capturing, and incident-analyzing Grafana dashboards whose datasource is Huawei Cloud LTS over vLLM/ModelArts engine logs. 端到端工作流：构建、编辑、截图、以及基于华为云 LTS（vLLM/ModelArts 引擎日志）数据源的 Grafana 看板做故障分析。

Chinese version: [README_zh.md](README_zh.md). 中文版见 README_zh.md。

## What it covers / 覆盖内容

- LTS query semantics over pre-aggregated metric lines (Engine ticks, fleet throughput, KV/prompt/gen). LTS 查询语义（Engine 指标行、集群吞吐、KV/提示/生成指标）。
- Per-engine calibers and the sum-of-per-engine-max form. 每引擎口径与"每引擎 max 之和"形态。
- Dashboard build/edit workflows over Grafana's DB. 基于 Grafana 数据库的看板构建/编辑流程。
- Capture of collapsed-row dashboards (render API + manual). 折叠行看板的截图（渲染 API + 手动）。
- Preemption / slow-TPOT incident analysis from metric-line signatures. 基于指标行特征增量的事故分析（抢占/慢 TPOT）。
- No credentials or environment identifiers in-repo. 仓库内不含凭据与环境标识。

## Requirements / 环境要求

- Grafana with the Huawei LTS datasource plugin (`hw-hws-lts-grafana-datasource-plugin`). 含华为 LTS 数据源插件的 Grafana。
- vLLM/ModelArts engine logs streaming into LTS with Engine metric lines. vLLM/ModelArts 引擎日志以 Engine 指标行形式流入 LTS。
- No credentials committed; source them at run time. 不提交凭据，运行时获取。

## Related skills / 关联技能

- `grafana-dashboards` — general Grafana setup/auth/renderer. 通用 Grafana 安装/鉴权/渲染。
- `grafana-dashboard-dev` — the portable engineering workflow this LTS layer extends. 本 LTS 层所扩展的可移植工程工作流。

## License / 许可证

MIT
