---
name: vllm-lts-ops-dashboard
description: "Build and analyze Grafana dashboards over vLLM LTS logs."
version: 0.2.1
author: Hermes Agent
license: MIT
platforms: [linux]
metadata:
  hermes:
    editorial_name: vLLM LTS Ops Dashboard
    editorial_description: Build Grafana dashboards over Huawei Cloud LTS logs for vLLM/ModelArts engine O&M. LTS query semantics, per-engine metric aggregation rules, dashboard build/edit workflows, PNG capture of collapsed-row dashboards, and preemption/TPOT incident analysis from metric-line signatures.
    tags: [grafana, lts, vllm, modelarts, observability, huawei-cloud]
    related_skills: [grafana-dashboards]
    requires_tools: [terminal, read_file, write_file, patch]
---

# vLLM LTS Ops Dashboard (Grafana × Huawei LTS)

End-to-end workflow for building, editing, capturing, and incident-analyzing Grafana dashboards whose datasource is Huawei Cloud LTS over vLLM/ModelArts engine logs. The log stream carries PRE-AGGREGATED metric lines (one `Engine` line per engine every ~10 s with running/waiting/kv_cache/prompt_throughput/gen_throughput) plus gateway access lines — NOT raw engine stdout. Extends `grafana-dashboards` (general Grafana setup, auth, renderer); this skill holds the LTS/vLLM-specific layer. No credentials or environment identifiers here — source them at run time (provisioning files, memory, the user).

## Output Locale / 输出语言

This skill produces dashboards and incident-triage conclusions for a human reader. Written reasoning follows the caller's language; SQL, LTS field names, API paths, and file names stay English-only, and technical terms (TPOT, kv_cache, prompt_throughput, PD-split, RECOMPUTE) are never translated. Frontmatter and body stay English-primary as an internal engineering skill; the bilingual pair lives in `README.md` + `README_zh.md`.

## When to Use
- Building/editing dashboards over an LTS datasource (`hw-hws-lts-grafana-datasource-plugin`), appName/podName-style structured labels.
- Fleet-total throughput panels over multi-engine (multi-pod) deployments.
- Scripted edits to dashboards stored in Grafana's DB (no provisioning files).
- PNG captures of dashboards whose rows default-collapsed.
- TPOT/preemption incident triage from engine metric lines.

## Don't Use For
- General Grafana auth/provisioning/renderer install — `grafana-dashboards`.
- LiteLLM Postgres/Prometheus analysis — `grafana-dashboards` references.

## LTS Query Semantics (the core)
1. **Metric lines are TICKS, not samples.** ~30 lines per engine per 5 m bucket; `sum()` over a gauge/rate column inflates 10–200×. `max()` for bounded per-engine gauges, `avg()` for per-engine ratios, `count(*)` for volume.
2. **No aggregate-of-aggregate in one SELECT**: `sum(max(col))` → 400 `LTS.2801 Cannot nest aggregations`. But a GROUPED subquery flattened to one value column WORKS:
```sql
select sum(v) as "<series>" from (
  select max(<col>) as v, <engine>, round(__time/$bucket)*$bucket as __time
  where appName LIKE '%<stable-deployment-id>%'
  group by <engine>, __time) group by __time order by __time limit 10000
```
   Fleet throughput MUST use this sum-of-per-engine-max form — a bare `max()` takes one engine's peak and under-reports by up to (n−1)/n. Title such panels `SUM of per-engine max (approx)`. Verify: compute per-engine max separately, assert sum ≈ Σ within ~1%. PD-split deployments (prefill/decode roles) log as separate role curves — fleet total = roleA + roleB, never merge.
3. `appName` filter: STABLE deployment-id substring, never pod names (pods rotate; history vanishes). `round(__time/$bucket)*$bucket as __time` preserves the time field; `date_trunc` drops it.
4. **`content` (raw text) is NOT selectable** — only structured fields. Probe candidate fields with `select max(<field>) ...`; unresolved column → 400 `Column '<x>' cannot be resolved` = absent.
5. **Frame parse order**: `/api/ds/query` returns fields as `[value, time]` (value FIRST) for LTS group-by queries — index by field NAME, never position.
6. To run an LTS query outside a panel, copy a working target object VERBATIM from the dashboard JSON (`groupId`, `streamId`, `queryText`, `xColumn`, `yColumn`, `rawQuery: true`) into `POST /api/ds/query` — the plugin needs its own field set, not rawSql. Get it via `GET /api/dashboards/uid/<uid>` and walking panels RECURSIVELY (row panels nest children in `row.panels`; a top-level-only walk finds 0 targets).
7. **Rate limit**: LTS front-door (APIGW) throttles per AK at low double-digit req/s, shared. Probe loops need `sleep ~1.3 s` between queries; mass-expanding collapsed rows → 429s → empty panels. Wait 30–60 s before calling panels broken. Disable auto-refresh (`refresh: ""`) for expensive dashboards.

## Incident Analysis from Metric Lines (preemption / slow TPOT)
- **What the stream does NOT contain**: vLLM engine-internal lines (`Sequence group ... preempted by PreemptionMode.RECOMPUTE`, swap warnings) are INFO/WARNING stdout that never reached LTS — keyword probes for preempt/recompute/swap (EN+ZH) return 0, and Engine lines carry NO preempted count field. Absence proves nothing about actual preemption.
- **Signature analysis instead** — align HOURLY series of kv_cache/running/waiting/prompt_throughput and scan for: kv near saturation + waiting pileup, followed by kv DROP ≥20 pts + prompt-throughput SPIKE ≥3× median (preempted sequences re-prefilling) with flat gen throughput. Zero such windows ⇒ preemption unlikely; pair with gateway-side per-request TPOT (slow only for requests batched WITH a giant prefill, self-healing after) — that is continuous-batching interference, not preemption, and the fix is different (chunked prefill / queue key isolation vs more KV memory).
- Deciding evidence: preemption requires KV pool exhaustion — if kv_cache never approaches ~100% and waiting stays ~0, rule it out.
- Precise preempted counts need the ModelArts gateway `/metrics` (`vllm:preempted_seqs_total`, needs API key) or raising the LTS ingest level to capture engine INFO/WARNING stdout.
- Retention: LTS dashboards here held ~31 d of Engine lines — check the daily-count lookback before promising historical coverage of an old incident.

## Dashboard Build & Edit Workflow
1. `GET /api/dashboards/uid/<uid>` → `{dashboard, version: N}`; edit; `POST /api/dashboards/db` with `{"dashboard": <keep version N>, "overwrite": true}`. Re-GET and assert: version bumped, rewritten queries present, old-pattern count 0.
2. Bulk query rewrites: load JSON once → ALL edits → save once; assert each replacement rule's hit count > 0; run ≥1 rewritten query through `/api/ds/query` before pushing.
3. Auth: anonymous Viewer reads; writes need admin (cookie from `POST /login` if Basic auth 401s — cookies outlive password resets). Never hardcode credentials in /tmp scripts.
4. Useful panel patterns: Running/Waiting sequences (sum of per-engine `max_by(col,__time)` per bucket), KV usage % (max + avg variants), completions volume (`"POST /v1" | count(*)`), active nodes (`count(distinct appName)`).

## Capturing Collapsed-Row Dashboards
### Render API
`GET /render/d/<uid>/<slug>?orgId=1&from=<epoch_ms>&to=<epoch_ms>&var-<var>=<val>&width=1720&height=<H>&scale=1&kiosk&timeout=60000` — slug REQUIRED (uid-only 404s); trust `/render/health` not the plugin flag. `height` is a TRUNCATING viewport, ~300 px taller than a Chrome `--screenshot` layout — never transfer a validated height between methods; step up empirically (1800 truncated one bottom table; 2200 was complete) and verify the bottom panel in the PNG. Epoch: `date -d '<wall time> <IANA tz>' +%s%3N`. Collapsed rows do NOT expand in render output.
### Manual capture (user-driven browser)
- Modern Chrome binds `--remote-debugging-port` to 127.0.0.1 regardless of `--remote-debugging-address` — a `:9222` debug-frontend URL is unreachable from other machines; NEVER hand one to the user.
- Chrome refuses a debug port on the DEFAULT user-data-dir; a capture Chrome needs its OWN profile dir, else a second launch just forwards the URL to the existing session and exits.
- Active-tab full-page shot: `/json/list` → match focused-window title (xdotool) or take the only/last http tab → WebSocket CDP → wait network-quiet + no spinner (≤60 s, respects rate limits) → `Page.captureScreenshot {captureBeyondViewport: true, fromSurface: true}`.
- UX: a GLOBAL HOTKEY + desktop notification beats in-page buttons ("can't find the button").
- If the user already has their own screenshot tool — stop building one, prefer theirs, and tear down everything you added (hotkey, icons, scripts, extra profiles, test outputs); verify the teardown.

## Verification
- [ ] Rewritten LTS query matches independent per-engine computation (~1%).
- [ ] Dashboard re-GET: version bumped; new queries present; forbidden patterns 0; row/geometry audited.
- [ ] Capture PNG: bottom panel complete; empty panels re-shot after a rate-limit wait.
- [ ] Incident analysis: signature scan run over the full retention window; absence-of-keyword never used as proof of absence-of-preemption.
- [ ] No process left depending on the agent session; user-visible artifacts delivered or rolled back.
