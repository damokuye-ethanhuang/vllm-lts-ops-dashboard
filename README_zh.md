# vLLM LTS Ops Dashboard / vLLM LTS 运维看板

端到端工作流：构建、编辑、截图、以及基于华为云 LTS（vLLM/ModelArts 引擎日志）数据源的 Grafana 看板做故障分析。English primary: [README.md](README.md)。

## 覆盖内容

- LTS 查询语义（Engine 指标行、集群吞吐、KV/提示/生成指标）。
- 每引擎口径与"每引擎 max 之和"形态。
- 基于 Grafana 数据库的看板构建/编辑流程。
- 折叠行看板的截图（渲染 API + 手动）。
- 基于指标行特征增量的事故分析（抢占/慢 TPOT）。
- 仓库内不含凭据与环境标识。

## 环境要求

- 含华为 LTS 数据源插件（hw-hws-lts-grafana-datasource-plugin）的 Grafana。
- vLLM/ModelArts 引擎日志以 Engine 指标行形式流入 LTS。
- 不提交凭据，运行时获取。

## 关联技能

- grafana-dashboards：通用 Grafana 安装/鉴权/渲染。
- grafana-dashboard-dev：本 LTS 层所扩展的可移植工程工作流。

## 许可证

MIT
