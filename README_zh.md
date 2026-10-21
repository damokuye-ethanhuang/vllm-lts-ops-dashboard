# vLLM LTS 运维看板

基于华为云 LTS（vLLM/ModelArts 引擎日志）数据源的 Grafana 看板端到端工作流：构建、编辑、截图与事故分析。English version: [README.md](README.md)。

## 覆盖内容

- 面向预聚合指标行的 LTS 查询语义（Engine 心跳行、集群吞吐、KV cache / 提示词 / 生成吞吐）。
- 每引擎指标的正确聚合方式，包括集群总量所需的"各引擎 max 之和"形态。
- 基于 Grafana 数据库的看板构建与编辑流程。
- 默认折叠行看板的 PNG 截图（渲染 API 与手动截图两条路）。
- 基于指标行特征的抢占与慢 TPOT 事故分析。

仓库内不存放任何凭据或环境标识。

## 环境要求

- 安装了华为 LTS 数据源插件（`hw-hws-lts-grafana-datasource-plugin`）的 Grafana。
- vLLM/ModelArts 引擎日志以 Engine 指标行的形式持续写入 LTS。

## 关联技能

- `grafana-dashboards`：Grafana 通用安装、鉴权与渲染。
- `grafana-dashboard-dev`：本 LTS 层所扩展的可移植工程工作流。

## 许可证

MIT
