# Hint Benchmark Runbook

Compact command table for the Agentic Hint Benchmark Suite.

Run these commands on the EC2 machine from the repo root:

```bash
cd ~/agentic_hardware
```

From the Mac, SSH with the configured EC2 host:

```bash
cd /Users/oluwolejaiyeoba/Documents/GitHub/agentic_hardware
bash -lc 'source aws/config.sh && ssh $(ssh_opts hintbench) "$EC2_USER@${SERVERS[0]}"'
cd ~/agentic_hardware
```

## Table Of Contents

- [Claude Code](#claude-code)
- [Qwen Code](#qwen-code)
- [Pi Agent Harness](#pi-agent-harness)
- [OpenClaw](#openclaw)
- [OpenCode](#opencode)
- [Deep Agents](#deep-agents)
- [DeepSeek Harness](#deepseek-harness)
- [Codex](#codex)
- [NeMo Agent Toolkit / NAT](#nemo-agent-toolkit--nat)
- [Hermes Agent](#hermes-agent)
- [Harness Scope Accounting](#harness-scope-accounting)
- [Unobserved Hint Experiment Backlog](#unobserved-hint-experiment-backlog)
- [Missing Or Blocked Today](#missing-or-blocked-today)
- [PPTX Signal Inventory](#pptx-signal-inventory)
- [Inspect Results](#inspect-results)

## Where Attached Legend

| Value | Meaning |
| --- | --- |
| Request | Applies to one outgoing model request. |
| Task | Applies to one task or workflow run. |
| Session | Applies to a session, so many requests in that session may carry it. |
| Configuration | Comes from a client/profile setting, so any run using that setting may carry it. |

## Source Lane Legend

| Value | Meaning |
| --- | --- |
| Native Claude Code | The real Claude Code CLI emitted the signal. |
| Claude Code + Provider | Claude Code was used, but provider routing/config may have produced the signal. |
| Native Qwen Code | The real Qwen Code CLI emitted or carried the signal at the request boundary. |
| Qwen Code + Provider Config | Qwen Code carried provider/model configuration such as `extra_body`, custom headers, or cache-control settings. |
| Native Pi Agent Harness | The real Pi Agent Harness CLI emitted or carried the signal at the request boundary. |
| Pi Agent Harness + Provider Config | Pi carried provider/model configuration such as prompt-cache retention, cache-control markers, or session-affinity headers. |
| Native OpenClaw | The real OpenClaw CLI emitted or carried the signal at the request boundary. |
| OpenClaw + Provider Config | OpenClaw carried provider/model configuration such as service-tier, provider cache, cached-WebSocket, or namespace settings. |
| Native OpenCode | The real OpenCode CLI emitted or carried the signal at the request boundary. |
| OpenCode + Provider Config | OpenCode carried provider or plugin cache metadata such as provider cache identity, plugin namespace, or usage stats. |
| Native Deep Agents | The real Deep Agents harness emitted or carried the signal at the request boundary. |
| Deep Agents + Provider Middleware | Deep Agents carried provider or middleware cache/QoS metadata. |
| Native DeepSeek Harness | The real DeepSeek Harness emitted or carried the signal at the request boundary. |
| DeepSeek Harness + Provider | DeepSeek Harness relied on provider-managed prefix cache, TTL, pinning, or cache-read feedback. |
| Native Codex | The real Codex client emitted or carried the signal at the request boundary. |
| Codex + Provider Config | Codex carried provider prompt-cache metadata such as `prompt_cache_key`, `service_tier`, WebSocket prewarm, or cached-token usage. |
| Native Hermes Agent | The real Hermes Agent CLI emitted or carried the signal at the request boundary. |
| Hermes Agent + Provider Config | Hermes carried provider/model configuration such as `service_tier`, `extra_body`, headers, or prompt-cache settings. |
| Native NAT Workflow | The real NeMo/NAT transport emitted the signal from workflow or transport settings. |
| NAT Pass-through | The signal was supplied before or around NAT, and NAT preserved it at the boundary. |
| Mixed NAT | The run intentionally combines NAT workflow signals and NAT pass-through signals. |

## Claude Code

<table>
<colgroup>
<col width="12%" style="width: 12%;">
<col width="18%" style="width: 18%;">
<col width="12%" style="width: 12%;">
<col width="25%" style="width: 25%;">
<col width="15%" style="width: 15%;">
<col width="16%" style="width: 16%;">
<col width="9%" style="width: 9%;">
<col width="8%" style="width: 8%;">
</colgroup>
<thead>
<tr>
<th>Run</th>
<th>Plain Purpose</th>
<th>Source Lane</th>
<th>Command</th>
<th>Signals Observed Today</th>
<th>When It Appears</th>
<th>Where Attached</th>
<th>Evidence</th>
</tr>
</thead>
<tbody>
<tr>
<td>Claude native baseline</td>
<td>Confirm Claude emits no benchmark hints when knobs are off.</td>
<td>Native Claude Code</td>
<td>

```bash
RUN_ID="claude_baseline_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness claude_code \
  --knob-profile baseline \
  --claude-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li>none</li>
</ul>
</td>
<td>No benchmark hint knobs are enabled: <code>--knob-profile baseline</code>.</td>
<td>Request</td>
<td>Native Claude Code request-boundary control case.</td>
</tr>
<tr>
<td>All Claude native request-boundary probes</td>
<td>Produce all Claude Code signals observed today in one run.</td>
<td>Native Claude Code</td>
<td>

```bash
RUN_ID="claude_all_request_boundary_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness claude_code \
  --knob-profile all_request_boundary \
  --claude-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>system.*.cache_control.type="ephemeral"</code></li>
<li><code>messages.*.content.*.cache_control.type="ephemeral"</code></li>
<li><code>cache_control.ttl="1h"</code></li>
<li><code>anthropic-beta: fast-mode-2026-02-01</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile all_request_boundary</code>; observed signals come from stable-context cache probes, <code>claude_fast_mode_setting</code> with <code>fastMode=true</code>, <code>ENABLE_PROMPT_CACHING_1H=1</code>, and <code>FORCE_PROMPT_CACHING_5M=1</code>.</td>
<td>Configuration</td>
<td>Native Claude Code request-boundary capture.</td>
</tr>
<tr>
<td>Claude native QoS probes</td>
<td>Produce the Claude fast-mode header.</td>
<td>Native Claude Code</td>
<td>

```bash
RUN_ID="claude_qos_only_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness claude_code \
  --knob-profile qos_only \
  --claude-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>anthropic-beta: fast-mode-2026-02-01</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile qos_only</code>, including scenario <code>claude_fast_mode_setting</code>.</td>
<td>Configuration</td>
<td>Native Claude Code request-boundary capture.</td>
</tr>
<tr>
<td>Claude native cache probes</td>
<td>Produce Claude prompt-cache markers and TTL.</td>
<td>Native Claude Code</td>
<td>

```bash
RUN_ID="claude_cache_only_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness claude_code \
  --knob-profile cache_only \
  --claude-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>system.*.cache_control.type="ephemeral"</code></li>
<li><code>messages.*.content.*.cache_control.type="ephemeral"</code></li>
<li><code>cache_control.ttl="1h"</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile cache_only</code>, including stable-prefix cache scenarios plus <code>claude_provider_retention_1h</code> with <code>ENABLE_PROMPT_CACHING_1H=1</code> and <code>claude_provider_retention_5m</code> with <code>FORCE_PROMPT_CACHING_5M=1</code>.</td>
<td>Configuration</td>
<td>Native Claude Code request-boundary capture.</td>
</tr>
<tr>
<td>Claude native prewarm probe</td>
<td>Send a request that warms a cacheable prompt block.</td>
<td>Native Claude Code</td>
<td>

```bash
RUN_ID="claude_prewarm_only_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness claude_code \
  --knob-profile prewarm_only \
  --claude-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>system.*.cache_control.type="ephemeral"</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile prewarm_only</code> with scenario <code>claude_prewarm_like_probe</code>.</td>
<td>Request</td>
<td>Native Claude Code request-boundary capture.</td>
</tr>
<tr>
<td>Claude native fast mode only</td>
<td>Produce only the Claude fast-mode header.</td>
<td>Native Claude Code</td>
<td>

```bash
RUN_ID="claude_fast_mode_setting_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness claude_code \
  --scenarios claude_fast_mode_setting \
  --claude-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>anthropic-beta: fast-mode-2026-02-01</code></li>
</ul>
</td>
<td>Runs scenario <code>claude_fast_mode_setting</code>, which passes Claude settings with <code>fastMode=true</code>.</td>
<td>Configuration</td>
<td>Native Claude Code request-boundary capture.</td>
</tr>
<tr>
<td>Claude native 1h cache TTL</td>
<td>Produce Claude cache control with a 1-hour TTL.</td>
<td>Native Claude Code</td>
<td>

```bash
RUN_ID="claude_provider_retention_1h_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness claude_code \
  --scenarios claude_provider_retention_1h \
  --claude-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>cache_control.type="ephemeral"</code></li>
<li><code>cache_control.ttl="1h"</code></li>
</ul>
</td>
<td>Runs scenario <code>claude_provider_retention_1h</code> with <code>ENABLE_PROMPT_CACHING_1H=1</code>.</td>
<td>Configuration</td>
<td>Native Claude Code request-boundary capture using `ENABLE_PROMPT_CACHING_1H=1`.</td>
</tr>
<tr>
<td>Claude native 5m cache control</td>
<td>Produce Claude cache control with default 5-minute retention.</td>
<td>Native Claude Code</td>
<td>

```bash
RUN_ID="claude_provider_retention_5m_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness claude_code \
  --scenarios claude_provider_retention_5m \
  --claude-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>cache_control.type="ephemeral"</code></li>
</ul>
</td>
<td>Runs scenario <code>claude_provider_retention_5m</code> with <code>FORCE_PROMPT_CACHING_5M=1</code>.</td>
<td>Configuration</td>
<td>Native Claude Code request-boundary capture using `FORCE_PROMPT_CACHING_5M=1`.</td>
</tr>
</tbody>
</table>

## Qwen Code

The current Qwen suite is request-boundary only. It proves Qwen Code can carry
configured provider/body/header/cache fields. It does not prove a real provider
or SGLang consumed them.

<table>
<colgroup>
<col width="12%" style="width: 12%;">
<col width="18%" style="width: 18%;">
<col width="12%" style="width: 12%;">
<col width="25%" style="width: 25%;">
<col width="15%" style="width: 15%;">
<col width="16%" style="width: 16%;">
<col width="9%" style="width: 9%;">
<col width="8%" style="width: 8%;">
</colgroup>
<thead>
<tr>
<th>Run</th>
<th>Plain Purpose</th>
<th>Source Lane</th>
<th>Command</th>
<th>Signals Observed Today</th>
<th>When It Appears</th>
<th>Where Attached</th>
<th>Evidence</th>
</tr>
</thead>
<tbody>
<tr>
<td>All Qwen request-boundary probes</td>
<td>Produce every Qwen signal we can observe today in one run.</td>
<td>Qwen Code + Provider Config</td>
<td>

```bash
RUN_ID="qwen_all_request_boundary_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness qwen_code \
  --knob-profile all_request_boundary \
  --qwen-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>service_tier="priority"</code></li>
<li><code>agentic_hints.priority_class="urgent"</code></li>
<li><code>x-hintbench-cache-namespace</code></li>
<li><code>cacheRetention="1h"</code></li>
<li><code>system.*.cache_control.type="ephemeral"</code></li>
<li><code>system.*.cache_control.ttl="1h"</code></li>
<li><code>agentic_hints.eviction_priority="high"</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile all_request_boundary</code>, combining OpenAI-compatible extra-body/header probes and the Anthropic-compatible cache-control probe.</td>
<td>Configuration</td>
<td>Native Qwen Code request-boundary capture. Current artifact: <code>qwen_native_full_20260921_local</code>.</td>
</tr>
<tr>
<td>Qwen native baseline</td>
<td>Confirm Qwen emits no benchmark hints when knobs are off.</td>
<td>Native Qwen Code</td>
<td>

```bash
RUN_ID="qwen_baseline_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness qwen_code \
  --knob-profile baseline \
  --qwen-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li>none</li>
</ul>
</td>
<td>No Qwen benchmark hint knobs are enabled: <code>--knob-profile baseline</code>.</td>
<td>Request</td>
<td>Native Qwen Code request-boundary control case.</td>
</tr>
<tr>
<td>Qwen provider QoS extra body</td>
<td>Carry provider QoS metadata through Qwen's OpenAI-compatible request body.</td>
<td>Qwen Code + Provider Config</td>
<td>

```bash
RUN_ID="qwen_qos_only_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness qwen_code \
  --knob-profile qos_only \
  --qwen-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>service_tier="priority"</code></li>
<li><code>agentic_hints.priority_class="urgent"</code></li>
</ul>
</td>
<td>Runs scenario <code>qwen_provider_qos_extra_body</code>, whose Qwen <code>generationConfig.extra_body</code> supplies provider metadata.</td>
<td>Configuration</td>
<td>Provider/config carried by Qwen Code, not organic priority inference.</td>
</tr>
<tr>
<td>Qwen cache signals</td>
<td>Produce Qwen cache-retention and cache-control markers.</td>
<td>Qwen Code + Provider Config</td>
<td>

```bash
RUN_ID="qwen_cache_only_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness qwen_code \
  --knob-profile cache_only \
  --qwen-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>cacheRetention="1h"</code></li>
<li><code>system.*.cache_control.type="ephemeral"</code></li>
<li><code>system.*.cache_control.ttl="1h"</code></li>
<li><code>anthropic-beta</code> cache TTL marker</li>
</ul>
</td>
<td>Runs OpenAI-compatible retention and Anthropic-compatible cache-control scenarios. The repeated-prefix cache-key probe is optional and did not expose a literal cache-key field today.</td>
<td>Configuration</td>
<td>Request-boundary cache metadata only; real provider cache-hit feedback is separate.</td>
</tr>
<tr>
<td>Qwen pass-through probes</td>
<td>Carry namespace and eviction metadata through Qwen configuration.</td>
<td>Qwen Code + Provider Config</td>
<td>

```bash
RUN_ID="qwen_passthrough_only_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness qwen_code \
  --knob-profile passthrough_only \
  --qwen-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>x-hintbench-cache-namespace</code></li>
<li><code>agentic_hints.eviction_priority="high"</code></li>
</ul>
</td>
<td>Runs namespace-header and eviction-metadata scenarios. These fields are supplied by benchmark config and preserved by Qwen.</td>
<td>Configuration</td>
<td>Pass-through/config evidence, not native Qwen scheduling or eviction policy.</td>
</tr>
</tbody>
</table>

## Pi Agent Harness

The current Pi suite has native request-boundary evidence from the real Pi CLI
via `npx -y @earendil-works/pi-coding-agent@latest`. Against
`presentation/Harness Signal Tables As-Is.pptx`, Pi has 6 supported or
conditional cache/provider signals and 10 unsupported focused signals. This run
observed 5 of the 6 supported or conditional signals at the request boundary.

<table>
<colgroup>
<col width="12%" style="width: 12%;">
<col width="18%" style="width: 18%;">
<col width="12%" style="width: 12%;">
<col width="25%" style="width: 25%;">
<col width="15%" style="width: 15%;">
<col width="16%" style="width: 16%;">
<col width="9%" style="width: 9%;">
<col width="8%" style="width: 8%;">
</colgroup>
<thead>
<tr>
<th>Run</th>
<th>Plain Purpose</th>
<th>Source Lane</th>
<th>Command</th>
<th>Signals Observed Today</th>
<th>When It Appears</th>
<th>Where Attached</th>
<th>Evidence</th>
</tr>
</thead>
<tbody>
<tr>
<td>All Pi request-boundary probes</td>
<td>Produce every Pi request-boundary signal observed today in one run.</td>
<td>Pi Agent Harness + Provider Config</td>
<td>

```bash
RUN_ID="pi_all_request_boundary_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness pi_agent_harness \
  --knob-profile all_request_boundary \
  --pi-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>prompt_cache_key</code></li>
<li><code>x-session-affinity</code></li>
<li><code>x-hintbench-cache-namespace</code></li>
<li><code>prompt_cache_retention="24h"</code></li>
<li><code>messages.*.cache_control.type="ephemeral"</code></li>
<li><code>messages.*.cache_control.ttl="1h"</code></li>
<li>absence of literal <code>cache_pinning</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile all_request_boundary</code>, combining provider prompt-cache, namespace, long-retention, cache-control, and pinning-negative probes.</td>
<td>Configuration</td>
<td>Native Pi request-boundary capture. Current artifact: <code>pi_native_request_boundary_rescored_20260921</code>.</td>
</tr>
<tr>
<td>Pi native baseline</td>
<td>Confirm Pi emits no benchmark cache hints when knobs are off.</td>
<td>Native Pi Agent Harness</td>
<td>

```bash
RUN_ID="pi_baseline_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness pi_agent_harness \
  --knob-profile baseline \
  --pi-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li>none</li>
</ul>
</td>
<td>No Pi benchmark hint knobs are enabled: <code>--knob-profile baseline</code>.</td>
<td>Request</td>
<td>Native Pi request-boundary control case.</td>
</tr>
<tr>
<td>Pi cache probes</td>
<td>Produce Pi prompt-cache key, retention, cache-control, and pinning-negative evidence.</td>
<td>Pi Agent Harness + Provider Config</td>
<td>

```bash
RUN_ID="pi_cache_only_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness pi_agent_harness \
  --knob-profile cache_only \
  --pi-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>prompt_cache_key</code></li>
<li><code>prompt_cache_retention="24h"</code></li>
<li><code>cache_control.type="ephemeral"</code></li>
<li><code>cache_control.ttl="1h"</code></li>
</ul>
</td>
<td>Runs provider prompt-cache, long-retention, and pinning-negative scenarios.</td>
<td>Configuration</td>
<td>Request-boundary cache metadata only; real cache-hit feedback is separate.</td>
</tr>
<tr>
<td>Pi namespace probes</td>
<td>Show provider/account or session-affinity cache isolation metadata.</td>
<td>Pi Agent Harness + Provider Config</td>
<td>

```bash
RUN_ID="pi_namespace_only_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness pi_agent_harness \
  --knob-profile namespace_only \
  --pi-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>x-session-affinity</code></li>
<li><code>x-hintbench-cache-namespace</code></li>
</ul>
</td>
<td>Runs scenario <code>pi_provider_account_namespace</code>, with session affinity enabled in the provider extension.</td>
<td>Session</td>
<td>Provider/config carried by Pi and preserved at the request boundary.</td>
</tr>
</tbody>
</table>

## OpenClaw

The current OpenClaw suite has native request-boundary evidence from the real
OpenClaw CLI via `npx -y openclaw@latest`. Against
`presentation/Harness Signal Tables As-Is.pptx`, OpenClaw has 8 supported or
conditional cache/provider signals and 8 unsupported focused signals. The
current request-boundary run observed 1 of the 8 supported or conditional
signals: configured namespace header preservation.

Current artifact: `openclaw_native_request_boundary_20260921`.

<table>
<colgroup>
<col width="12%" style="width: 12%;">
<col width="18%" style="width: 18%;">
<col width="12%" style="width: 12%;">
<col width="25%" style="width: 25%;">
<col width="15%" style="width: 15%;">
<col width="16%" style="width: 16%;">
<col width="9%" style="width: 9%;">
<col width="8%" style="width: 8%;">
</colgroup>
<thead>
<tr>
<th>Run</th>
<th>Plain Purpose</th>
<th>Source Lane</th>
<th>Command</th>
<th>Signals Observed Today</th>
<th>When It Appears</th>
<th>Where Attached</th>
<th>Evidence</th>
</tr>
</thead>
<tbody>
<tr>
<td>All OpenClaw request-boundary probes</td>
<td>Run every OpenClaw request-boundary probe and report what actually appears.</td>
<td>OpenClaw + Provider Config</td>
<td>

```bash
RUN_ID="openclaw_all_request_boundary_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness openclaw \
  --knob-profile all_request_boundary \
  --openclaw-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>x-hintbench-cache-namespace</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile all_request_boundary</code>, combining provider QoS, provider/session namespace, provider cache config, cached-WebSocket/prewarm, and cache-pinning negative probes.</td>
<td>Session</td>
<td>Native OpenClaw request-boundary capture. Current artifact: <code>openclaw_native_request_boundary_20260921</code>.</td>
</tr>
<tr>
<td>OpenClaw native baseline</td>
<td>Confirm OpenClaw emits no benchmark cache or QoS hints when knobs are off.</td>
<td>Native OpenClaw</td>
<td>

```bash
RUN_ID="openclaw_baseline_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness openclaw \
  --knob-profile baseline \
  --openclaw-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li>none</li>
</ul>
</td>
<td>No OpenClaw benchmark hint knobs are enabled: <code>--knob-profile baseline</code>.</td>
<td>Request</td>
<td>Native OpenClaw request-boundary control case.</td>
</tr>
<tr>
<td>OpenClaw namespace probe</td>
<td>Show provider/session cache namespace metadata when configured.</td>
<td>OpenClaw + Provider Config</td>
<td>

```bash
RUN_ID="openclaw_namespace_only_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness openclaw \
  --knob-profile namespace_only \
  --openclaw-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>x-hintbench-cache-namespace</code></li>
</ul>
</td>
<td>Runs scenario <code>openclaw_provider_session_namespace</code>, whose provider config supplies a namespace header.</td>
<td>Session</td>
<td>Provider/config carried by OpenClaw and preserved at the request boundary.</td>
</tr>
<tr>
<td>OpenClaw cache candidate probes</td>
<td>Probe provider-managed cache key, retention, entry type, cached-WebSocket, and pinning-negative recipes.</td>
<td>OpenClaw + Provider Config</td>
<td>

```bash
RUN_ID="openclaw_cache_only_$(date +%Y%m%d_%H%M%S)"
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness openclaw \
  --knob-profile cache_only \
  --openclaw-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li>none observed in the current native request-boundary run</li>
</ul>
</td>
<td>Runs provider cache config, cached-WebSocket/prewarm, and cache-pinning negative probes.</td>
<td>Configuration</td>
<td>These remain provider-managed candidates today; the current OpenClaw request body did not expose literal cache fields.</td>
</tr>
</tbody>
</table>

## OpenCode

OpenCode appears on slides 7 and 12 of
`presentation/Harness Signal Tables As-Is.pptx`. The deck shows no scheduling
signals for OpenCode. It shows three cache/provider candidates: provider-managed
cache identity, provider or plugin namespace, and provider/plugin cache usage
feedback. OpenCode now has a manifest, scenarios, knobs, fixture validation,
and a provider/plugin configuration capture lane.

OpenCode signal accounting against `presentation/Harness Signal Tables As-Is.pptx`:

```text
focused serving-control signals for OpenCode: 16
deck-supported or conditional signals: 3
fixture/plumbing signals observed: 3
provider/plugin config signals observed: 2
native CLI signals observed: 0
supported/conditional signals not provider/native observed yet: 1
deck-unsupported signals: 13
```

| Run | Plain Purpose | Source Lane | Command | Signals Observed Today | When It Appears | Where Attached | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| OpenCode provider/plugin config full coverage | Capture the OpenCode provider/plugin config signals we can represent locally. | OpenCode + Provider Config | `python3 sglang_direct_kv/scripts/run_hint_benchmark.py --harness opencode --knob-profile full_coverage --opencode-provider-config-capture --run-id opencode_provider_config_20260922` | `helicone-cache-key`, namespace header | Runs all OpenCode scenarios, but skips real-provider-only usage feedback. | Configuration | Artifact: `opencode_provider_config_20260922`; provider/plugin config evidence, not native CLI proof. |
| OpenCode fixture full coverage | Validate every OpenCode signal recipe currently described by the deck. | OpenCode + Provider Config | `python3 sglang_direct_kv/scripts/run_hint_benchmark.py --harness opencode --knob-profile full_coverage --fixture-observations --run-id opencode_fixture_full_20260922` | `helicone-cache-key`, namespace header, `usage.cached_tokens` fixture shape | Runs all OpenCode scenarios from the manifest. | Configuration and runtime feedback | Fixture artifact: `opencode_fixture_full_20260922`; not native evidence. |
| OpenCode provider/plugin cache probe | Probe provider-managed cache key and provider/plugin namespace behavior. | OpenCode + Provider Config | `--harness opencode --knob-profile cache_only --opencode-provider-config-capture` | `helicone-cache-key`, namespace header | Runs provider-cache and plugin namespace scenarios. | Configuration | Provider/plugin config evidence, not native CLI proof. |
| OpenCode provider feedback probe | Probe whether OpenCode/provider/plugin usage stats expose cache-hit feedback. | OpenCode + Provider Config | `--harness opencode --knob-profile feedback_only --opencode-provider-config-capture` | none yet | Requires provider/plugin stats or usage output for real evidence. | Runtime feedback | Still missing; real provider/plugin required. |

## Deep Agents

Deep Agents appears on slides 6 and 11 of
`presentation/Harness Signal Tables As-Is.pptx`, and slide 3 says it was
already covered. This runbook now represents it explicitly, but this suite does
now contains a fresh Deep Agents manifest, scenario set, knob profiles, and
fixture validation, and a middleware/provider configuration capture lane. The
deck shows provider-dependent QoS plus middleware/provider cache signals.

Deep Agents signal accounting against `presentation/Harness Signal Tables As-Is.pptx`:

```text
focused serving-control signals for Deep Agents: 16
deck-supported or conditional signals: 8
fixture/plumbing signals observed: 8
provider/middleware config signals observed: 7
native CLI/library signals observed: 0
supported/conditional signals not provider/native observed yet: 1
deck-unsupported signals: 8
```

| Run | Plain Purpose | Source Lane | Command | Signals Observed Today | When It Appears | Where Attached | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Deep Agents middleware full coverage | Capture the Deep Agents provider/middleware signals we can represent locally. | Deep Agents + Provider Middleware | `python3 sglang_direct_kv/scripts/run_hint_benchmark.py --harness deep_agents --knob-profile full_coverage --deep-agents-middleware-capture --run-id deep_agents_middleware_20260922` | provider QoS, middleware cache key, namespace, TTL, type, custom eviction, pinning | Runs all Deep Agents scenarios, but skips real-provider trace feedback. | Configuration | Artifact: `deep_agents_middleware_20260922`; provider/middleware evidence, not native library/CLI proof. |
| Deep Agents fixture full coverage | Validate every Deep Agents signal recipe currently described by the deck. | Deep Agents + Provider Middleware | `python3 sglang_direct_kv/scripts/run_hint_benchmark.py --harness deep_agents --knob-profile full_coverage --fixture-observations --run-id deep_agents_fixture_full_20260922` | provider QoS, middleware cache key, namespace, TTL, type, eviction, pinning, trace feedback fixture shapes | Runs all Deep Agents scenarios from the manifest. | Configuration and runtime feedback | Fixture artifact: `deep_agents_fixture_full_20260922`; not native evidence. |
| Deep Agents provider QoS probe | Probe provider-dependent speed/QoS routing. | Deep Agents + Provider Middleware | `--harness deep_agents --knob-profile qos_only --deep-agents-middleware-capture` | `provider.qos_tier` | Runs a provider QoS scenario. | Provider | Provider/middleware config evidence. |
| Deep Agents middleware cache probes | Probe middleware/provider cache identity, namespace, TTL, cache type, custom eviction, and pinning. | Deep Agents + Provider Middleware | `--harness deep_agents --knob-profile cache_only --deep-agents-middleware-capture` | middleware cache key, namespace, TTL, type, eviction, pinning | Runs middleware cache scenarios. | Configuration | Provider/middleware config evidence. |
| Deep Agents provider trace feedback | Probe provider trace data for cache-hit feedback. | Deep Agents + Provider Middleware | `--harness deep_agents --knob-profile feedback_only --deep-agents-middleware-capture` | none yet | Requires provider trace or metrics after repeated cacheable requests for real evidence. | Runtime feedback | Still missing; real provider trace required. |

## DeepSeek Harness

DeepSeek Harness appears on slides 6 and 11 of
`presentation/Harness Signal Tables As-Is.pptx`. The deck shows no scheduling
signals. It shows four provider-managed cache candidates: automatic prefix
matching, provider-managed TTL, `cacheReadTokens` feedback, and provider-managed
pinning. DeepSeek Harness now has a manifest, scenarios, knobs, fixture
validation, and a provider-capability capture lane.

DeepSeek Harness signal accounting against `presentation/Harness Signal Tables As-Is.pptx`:

```text
focused serving-control signals for DeepSeek Harness: 16
deck-supported or conditional signals: 4
fixture/plumbing signals observed: 4
provider capability signals observed: 3
native harness signals observed: 0
supported/conditional signals not provider/native observed yet: 1
deck-unsupported signals: 12
```

| Run | Plain Purpose | Source Lane | Command | Signals Observed Today | When It Appears | Where Attached | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DeepSeek provider capability full coverage | Capture provider-managed DeepSeek cache capability signals we can represent locally. | DeepSeek Harness + Provider | `python3 sglang_direct_kv/scripts/run_hint_benchmark.py --harness deepseek_harness --knob-profile full_coverage --deepseek-provider-capability-capture --run-id deepseek_provider_capability_20260922` | prefix cache policy, provider-managed TTL, provider-managed pinning | Runs all DeepSeek scenarios, but skips real-provider `cacheReadTokens` feedback. | Configuration | Artifact: `deepseek_provider_capability_20260922`; provider capability evidence, not native harness proof. |
| DeepSeek fixture full coverage | Validate every DeepSeek Harness signal recipe currently described by the deck. | DeepSeek Harness + Provider | `python3 sglang_direct_kv/scripts/run_hint_benchmark.py --harness deepseek_harness --knob-profile full_coverage --fixture-observations --run-id deepseek_harness_fixture_full_20260922` | prefix cache, provider TTL, provider pinning, `cacheReadTokens` fixture shapes | Runs all DeepSeek Harness scenarios from the manifest. | Configuration and runtime feedback | Fixture artifact: `deepseek_harness_fixture_full_20260922`; not native evidence. |
| DeepSeek provider cache probe | Probe automatic prefix matching, provider-managed TTL, and provider-managed pinning. | DeepSeek Harness + Provider | `--harness deepseek_harness --knob-profile cache_only --deepseek-provider-capability-capture` | prefix cache policy, provider TTL, provider pinning | Runs repeated stable-prefix request recipes. | Configuration | Provider capability evidence. |
| DeepSeek cache feedback probe | Probe `cacheReadTokens` or provider-equivalent cache feedback. | DeepSeek Harness + Provider | `--harness deepseek_harness --knob-profile feedback_only --deepseek-provider-capability-capture` | none yet | Requires real provider response or metrics output for real evidence. | Runtime feedback | Still missing; real provider required. |

## Codex

Codex appears on slides 5 and 10 of
`presentation/Harness Signal Tables As-Is.pptx`, and slide 3 says it was
already covered. This runbook now represents it explicitly, but this suite does
now contains a fresh Codex manifest, scenario set, knob profiles, and fixture
validation, and a provider-configuration capture lane. The deck shows provider
service tier, `prompt_cache_key`, prompt-cache bucketing,
WebSocket prewarm, and cached input-token usage. It also notes no explicit TTL.

Codex signal accounting against `presentation/Harness Signal Tables As-Is.pptx`:

```text
focused serving-control signals for Codex: 16
deck-supported or conditional signals: 5
fixture/plumbing signals observed: 5
provider config signals observed: 4
native Codex client signals observed: 0
supported/conditional signals not provider/native observed yet: 1
deck-unsupported signals: 11
```

| Run | Plain Purpose | Source Lane | Command | Signals Observed Today | When It Appears | Where Attached | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Codex provider config full coverage | Capture the Codex provider config signals we can represent locally. | Codex + Provider Config | `python3 sglang_direct_kv/scripts/run_hint_benchmark.py --harness codex --knob-profile full_coverage --codex-provider-config-capture --run-id codex_provider_config_20260922` | `service_tier`, `prompt_cache_key`, cache bucketing, WebSocket prewarm marker | Runs all Codex scenarios, but skips real-provider cached-token feedback. | Configuration | Artifact: `codex_provider_config_20260922`; provider config evidence, not native Codex CLI proof. |
| Codex fixture full coverage | Validate every Codex signal recipe currently described by the deck. | Codex + Provider Config | `python3 sglang_direct_kv/scripts/run_hint_benchmark.py --harness codex --knob-profile full_coverage --fixture-observations --run-id codex_fixture_full_20260922` | service tier, `prompt_cache_key`, cache bucketing, WebSocket prewarm, cached-token feedback fixture shapes | Runs all Codex scenarios from the manifest. | Configuration and runtime feedback | Fixture artifact: `codex_fixture_full_20260922`; not native evidence. |
| Codex provider QoS probe | Probe `service_tier` provider routing. | Codex + Provider Config | `--harness codex --knob-profile qos_only --codex-provider-config-capture` | `service_tier` | Runs a service-tier scenario. | Provider | Provider config evidence. |
| Codex prompt cache probe | Probe `prompt_cache_key`, cache bucketing, and WebSocket prewarm. | Codex + Provider Config | `--harness codex --knob-profile cache_only --codex-provider-config-capture` | `prompt_cache_key`, bucketing, WebSocket prewarm marker | Runs stable-prefix and prewarm scenarios. | Configuration | Provider config evidence. |
| Codex cache feedback probe | Probe cached input-token usage. | Codex + Provider Config | `--harness codex --knob-profile feedback_only --codex-provider-config-capture` | none yet | Requires real provider response or client usage output for real evidence. | Runtime feedback | Still missing; real provider/client usage output required. |

## NeMo Agent Toolkit / NAT

<table>
<colgroup>
<col width="12%" style="width: 12%;">
<col width="18%" style="width: 18%;">
<col width="12%" style="width: 12%;">
<col width="25%" style="width: 25%;">
<col width="15%" style="width: 15%;">
<col width="16%" style="width: 16%;">
<col width="9%" style="width: 9%;">
<col width="8%" style="width: 8%;">
</colgroup>
<thead>
<tr>
<th>Run</th>
<th>Plain Purpose</th>
<th>Source Lane</th>
<th>Command</th>
<th>Signals Observed Today</th>
<th>When It Appears</th>
<th>Where Attached</th>
<th>Evidence</th>
</tr>
</thead>
<tbody>
<tr>
<td>NAT baseline</td>
<td>Confirm NAT emits no hints when knobs are off.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_baseline_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --knob-profile baseline \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li>none</li>
</ul>
</td>
<td>No NAT benchmark hint knobs are enabled: <code>--knob-profile baseline</code>.</td>
<td>Request</td>
<td>Native NAT transport capture control case.</td>
</tr>
<tr>
<td>All NAT request-boundary signals</td>
<td>Produce every NAT signal we can observe today in one run.</td>
<td>Mixed NAT</td>
<td>

```bash
RUN_ID="nat_all_request_boundary_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --knob-profile all_request_boundary \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>priority</code></li>
<li><code>latency_sensitivity</code></li>
<li><code>osl</code></li>
<li><code>iat</code></li>
<li><code>total_requests</code></li>
<li><code>prefix_id</code></li>
<li><code>nvext.cache_control.ttl</code></li>
<li><code>nvext.cache_control.type</code></li>
<li><code>nvext.cache_salt</code></li>
<li><code>provider.qos_tier</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile all_request_boundary</code>, which combines scheduling, cache-control, namespace, and provider QoS scenarios.</td>
<td>Configuration</td>
<td>Native NAT `_DynamoTransport` request-boundary capture.</td>
</tr>
<tr>
<td>Scheduling signals only</td>
<td>Produce NAT priority and request-planning signals.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_scheduling_only_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --knob-profile scheduling_only \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>priority</code></li>
<li><code>latency_sensitivity</code></li>
<li><code>osl</code></li>
<li><code>iat</code></li>
<li><code>total_requests</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile scheduling_only</code>, including high/low priority, latency sensitivity, output length, cadence, and planned-count scenarios.</td>
<td>Task</td>
<td>Native NAT transport capture.</td>
</tr>
<tr>
<td>Cache signals only</td>
<td>Produce NAT cache reuse and cache-control signals.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_cache_only_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --knob-profile cache_only \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>prefix_id</code></li>
<li><code>nvext.cache_control.ttl</code></li>
<li><code>nvext.cache_control.type</code></li>
<li><code>priority</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile cache_only</code>, including prefix reuse, TTL, ephemeral cache control, first-only cache control, and eviction-priority intent.</td>
<td>Configuration</td>
<td>Native NAT transport capture.</td>
</tr>
<tr>
<td>Pass-through signals only</td>
<td>Preserve provider and session metadata through NAT.</td>
<td>NAT Pass-through</td>
<td>

```bash
RUN_ID="nat_passthrough_only_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --knob-profile passthrough_only \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>nvext.cache_salt</code></li>
<li><code>provider.qos_tier</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile passthrough_only</code>, including <code>nat_cache_namespace</code> and <code>nat_provider_qos</code>.</td>
<td>Session</td>
<td>Preserved through NAT transport; provider QoS is pass-through, not NAT-invented.</td>
</tr>
<tr>
<td>Priority high</td>
<td>Produce a high-priority NAT request.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_priority_high_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_priority_high \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>priority=100</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_priority_high</code>, whose workflow metadata sets <code>priority="high"</code>.</td>
<td>Task</td>
<td>Native NAT transport capture.</td>
</tr>
<tr>
<td>Priority low</td>
<td>Produce a low-priority NAT request.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_priority_low_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_priority_low \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>priority=2</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_priority_low</code>, whose workflow metadata sets <code>priority="low"</code>.</td>
<td>Task</td>
<td>Native NAT transport capture.</td>
</tr>
<tr>
<td>Latency sensitive</td>
<td>Produce a latency-sensitive NAT request.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_latency_sensitive_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_latency_sensitive \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>latency_sensitivity</code></li>
<li><code>priority</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_latency_sensitive</code>, whose workflow metadata sets <code>latency_sensitivity=100</code>.</td>
<td>Task</td>
<td>Native NAT transport capture.</td>
</tr>
<tr>
<td>Expected output length</td>
<td>Produce the expected output length hint.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_expected_output_length_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_expected_output_length \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>osl</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_expected_output_length</code>, whose workflow metadata sets <code>osl=128</code>.</td>
<td>Task</td>
<td>Native NAT transport capture from configured workload metadata.</td>
</tr>
<tr>
<td>Expected interarrival time</td>
<td>Produce the expected request spacing hint.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_expected_interarrival_time_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_expected_interarrival_time \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>iat</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_expected_interarrival_time</code>, whose workflow metadata sets <code>iat=750</code>.</td>
<td>Task</td>
<td>Native NAT transport capture from configured workload metadata.</td>
</tr>
<tr>
<td>Planned request count</td>
<td>Produce the planned request count hint.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_remaining_calls_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_remaining_calls \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>total_requests</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_remaining_calls</code>, whose workflow metadata sets <code>total_requests=4</code>.</td>
<td>Task</td>
<td>Native NAT transport capture.</td>
</tr>
<tr>
<td>Prefix reuse ID</td>
<td>Produce the reusable prefix/session ID hint.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_prefix_reuse_id_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_prefix_reuse_id \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>prefix_id</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_prefix_reuse_id</code>, whose workflow metadata sets a shared <code>prefix_id</code>.</td>
<td>Session</td>
<td>Native NAT transport capture.</td>
</tr>
<tr>
<td>Cache TTL</td>
<td>Produce a cache lifetime hint.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_cache_control_ttl_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_cache_control_ttl \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>nvext.cache_control.ttl</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_cache_control_ttl</code>, whose workflow metadata sets <code>cache_control.ttl="1s"</code>.</td>
<td>Request</td>
<td>Native NAT transport capture.</td>
</tr>
<tr>
<td>Ephemeral cache entry</td>
<td>Produce an ephemeral cache-control hint.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_cache_ephemeral_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_cache_ephemeral \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>nvext.cache_control.type="ephemeral"</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_cache_ephemeral</code>, whose workflow metadata sets <code>cache_control.type="ephemeral"</code>.</td>
<td>Request</td>
<td>Native NAT transport capture.</td>
</tr>
<tr>
<td>First-only cache control</td>
<td>Produce cache control only on the first repeated request.</td>
<td>Native NAT Workflow</td>
<td>

```bash
RUN_ID="nat_cache_control_first_only_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_cache_control_first_only \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>nvext.cache_control</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_cache_control_first_only</code>, whose workflow metadata sets <code>cache_control.mode="first_only"</code>.</td>
<td>Task</td>
<td>Native NAT transport capture.</td>
</tr>
<tr>
<td>Cache namespace</td>
<td>Produce a cache namespace hint.</td>
<td>NAT Pass-through</td>
<td>

```bash
RUN_ID="nat_cache_namespace_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_cache_namespace \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>nvext.cache_salt</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_cache_namespace</code>, whose client metadata supplies <code>nvext.cache_salt</code>.</td>
<td>Session</td>
<td>Pass-through preserved by NAT transport.</td>
</tr>
<tr>
<td>Provider QoS pass-through</td>
<td>Preserve provider QoS metadata through NAT.</td>
<td>NAT Pass-through</td>
<td>

```bash
RUN_ID="nat_provider_qos_$(date +%Y%m%d_%H%M%S)"
.venvs/nat_py311/bin/python sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness nemo_agent_toolkit \
  --scenarios nat_provider_qos \
  --nat-dynamo-transport-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>provider.qos_tier</code></li>
</ul>
</td>
<td>Runs scenario <code>nat_provider_qos</code>, whose provider metadata supplies <code>qos_tier</code>.</td>
<td>Configuration</td>
<td>Pass-through preserved by NAT transport; not NAT-invented.</td>
</tr>
</tbody>
</table>

## Hermes Agent

The current Hermes suite has request-boundary setup and fixture validation. A
native Hermes run requires a Hermes CLI on the host, for example
`HARNESS_HERMES_BIN=$HOME/agentic_hardware/.venvs/hermes_agent_py311/bin/hermes`.
Do not count fixture output as native Hermes evidence.

<table>
<colgroup>
<col width="12%" style="width: 12%;">
<col width="18%" style="width: 18%;">
<col width="12%" style="width: 12%;">
<col width="25%" style="width: 25%;">
<col width="15%" style="width: 15%;">
<col width="16%" style="width: 16%;">
<col width="9%" style="width: 9%;">
<col width="8%" style="width: 8%;">
</colgroup>
<thead>
<tr>
<th>Run</th>
<th>Plain Purpose</th>
<th>Source Lane</th>
<th>Command</th>
<th>Signals Targeted</th>
<th>When It Appears</th>
<th>Where Attached</th>
<th>Evidence</th>
</tr>
</thead>
<tbody>
<tr>
<td>All Hermes request-boundary probes</td>
<td>Produce every Hermes request-boundary signal recipe in one run.</td>
<td>Hermes Agent + Provider Config</td>
<td>

```bash
RUN_ID="hermes_all_request_boundary_$(date +%Y%m%d_%H%M%S)"
HARNESS_HERMES_BIN="$HOME/agentic_hardware/.venvs/hermes_agent_py311/bin/hermes" \
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness hermes_agent \
  --knob-profile all_request_boundary \
  --hermes-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>service_tier="priority"</code></li>
<li><code>agentic_hints.priority_class="urgent"</code></li>
<li><code>x-hintbench-cache-namespace</code></li>
<li><code>system.*.cache_control.type</code></li>
<li><code>system.*.cache_control.ttl="1h"</code></li>
<li>absence of literal <code>cache_pinning</code></li>
</ul>
</td>
<td>Runs <code>--knob-profile all_request_boundary</code>, combining provider QoS, namespace/isolation, prompt-cache TTL, and cache-pinning negative probes.</td>
<td>Configuration</td>
<td>Native Hermes Agent request-boundary capture once the Hermes CLI is installed. Local fixture check: <code>hermes_fixture_check</code>.</td>
</tr>
<tr>
<td>Hermes setup smoke</td>
<td>Verify the Hermes manifest, scenarios, and validator plumbing without claiming native evidence.</td>
<td>Hermes Agent + Provider Config</td>
<td>

```bash
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness hermes_agent \
  --knob-profile all_request_boundary \
  --fixture-observations \
  --run-id hermes_fixture_check
```

</td>
<td>
<ul>
<li>fixture-only validator rows</li>
<li>no unknown hints</li>
</ul>
</td>
<td>Use before native capture, or on machines without the Hermes CLI.</td>
<td>Configuration</td>
<td><code>fixture_plumbing_only</code>; not native harness evidence.</td>
</tr>
<tr>
<td>Hermes provider QoS</td>
<td>Carry service-tier metadata through Hermes provider config.</td>
<td>Hermes Agent + Provider Config</td>
<td>

```bash
RUN_ID="hermes_qos_only_$(date +%Y%m%d_%H%M%S)"
HARNESS_HERMES_BIN="$HOME/agentic_hardware/.venvs/hermes_agent_py311/bin/hermes" \
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness hermes_agent \
  --knob-profile qos_only \
  --hermes-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>service_tier="priority"</code></li>
<li><code>agentic_hints.priority_class="urgent"</code></li>
</ul>
</td>
<td>Runs scenario <code>hermes_provider_qos_request_overrides</code>, whose provider config supplies service-tier metadata.</td>
<td>Configuration</td>
<td>Provider/config evidence, not organic urgency inference.</td>
</tr>
<tr>
<td>Hermes cache probes</td>
<td>Probe prompt-cache TTL/type and provider-derived cache-key behavior.</td>
<td>Hermes Agent + Provider Config</td>
<td>

```bash
RUN_ID="hermes_cache_only_$(date +%Y%m%d_%H%M%S)"
HARNESS_HERMES_BIN="$HOME/agentic_hardware/.venvs/hermes_agent_py311/bin/hermes" \
python3 sglang_direct_kv/scripts/run_hint_benchmark.py \
  --harness hermes_agent \
  --knob-profile cache_only \
  --hermes-native-capture \
  --run-id "$RUN_ID" \
  --out-dir "sglang_direct_kv/artifacts/results/hint_benchmark/$RUN_ID"
```

</td>
<td>
<ul>
<li><code>system.*.cache_control.type</code></li>
<li><code>system.*.cache_control.ttl="1h"</code></li>
<li><code>model</code> as part of provider cache identity</li>
</ul>
</td>
<td>Runs prompt-cache TTL, repeated-prefix cache-key, and cache-pinning negative scenarios.</td>
<td>Configuration</td>
<td>Request-boundary cache metadata only; real provider cache-hit feedback is separate.</td>
</tr>
</tbody>
</table>

## Harness Scope Accounting

This table tracks the deck-level harness universe separately from signal-level
coverage. Use it when asking "how many harnesses are left?"

| Harness | Deck Status | Benchmark Status | Counted As Left? | Notes |
| --- | --- | --- | --- | --- |
| NeMo Agent Toolkit / NAT | In active deck tables | Native/request-boundary evidence complete for current suite | no | Current suite foundation. |
| Claude Code | In active deck tables | Native/request-boundary evidence complete for current suite; real-provider feedback still blocked | no | Feedback work remains, but the harness pass exists. |
| Qwen Code | In active deck tables | Native/request-boundary evidence complete for current suite | no | Provider/config signal pass exists. |
| Pi Agent Harness | Additional harness in deck | Native/request-boundary evidence complete for current suite | no | Current suite has signal accounting. |
| OpenClaw | Additional harness in deck | Native/request-boundary evidence exists; many signals still unobserved | no | Harness pass exists; omission experiments remain. |
| Hermes Agent | Additional harness in deck | Manifest/scenarios/fixtures exist; native CLI evidence pending | yes, if counting incomplete native evidence | Needs a host with working Hermes CLI. |
| OpenCode | Additional harness in deck | Provider/plugin config capture exists for 2 of 3 deck signals; native CLI proof still pending | no for provider-config coverage; yes for native CLI proof | Real-provider feedback still missing. |
| Deep Agents | In deck; slide 3 says already covered | Provider/middleware capture exists for 7 of 8 deck signals; native library/CLI proof still pending | no for middleware coverage; yes for native proof | Real-provider trace feedback still missing. |
| DeepSeek Harness | In deck scheduling/cache tables | Provider-capability capture exists for 3 of 4 deck signals | no for provider-capability coverage | Real-provider `cacheReadTokens` feedback still missing. |
| Dynamo | Slide 3 says already covered | Not represented as an active benchmark pass here | optional | Re-open only if we want fresh evidence. |
| Codex | Slide 3 says already covered | Provider-config capture exists for 4 of 5 deck signals; native Codex client proof still pending | no for provider-config coverage; yes for native Codex proof | Real-provider cached-token feedback still missing. |

## Unobserved Hint Experiment Backlog

This table is the future experiment queue. A row here means the signal has not
been observed in the current benchmark evidence for that harness, or it was
only observed through a weaker lane than the one we want. Use it to plan the
next round of omission-closing experiments.

| Harness | Hint Not Observed Or Not Fully Proven | Deck-Supported? | Current Reason | Next Experiment To Try | Requires Real Provider? |
| --- | --- | --- | --- | --- | --- |
| NAT | cache-hit runtime feedback | yes | Request-boundary capture cannot show backend cache hits or runtime reuse metrics. | Run a real backend cache-hit experiment and collect SGLang/NAT profiler or metrics output after repeated cacheable requests. | yes |
| NAT | literal `cache_pinning=true` | no | NAT 1.8.0 exposes first-only/ephemeral cache control, not a separate pinning field. | Recheck on a newer NAT path or treat FIRST_ONLY/ephemeral cache control as the observable pin-like behavior. | no |
| Claude Code | native `service_tier=auto` / `standard_only` body field | conditional | Tested Claude Code env-var paths did not forward a literal body field. | Try a Claude Code/provider configuration that explicitly forwards `ANTHROPIC_SERVICE_TIER` or `CLAUDE_CODE_SERVICE_TIER`, then rerun the service-tier scenarios. | no |
| Claude Code | literal cache key | conditional | Claude appears to use provider-derived exact-prefix matching without exposing a cache-key field. | Use a provider/API path that surfaces prompt-cache identity, or keep validating repeated-prefix behavior without expecting a literal key. | maybe |
| Claude Code | tool-level `cache_control` | conditional | Tool-heavy request had tools, but no `tools.*.cache_control` marker. | Build a stable-tool-definition scenario and inspect whether Claude Code/provider marks reusable tool definitions directly. | no |
| Claude Code | native `max_tokens=0` prewarm | conditional | Native Claude Code capture emitted cache control, but not a zero-token prewarm request. | Find or add a Claude Code path that can issue a prewarm-like request, then compare against the direct API max-token-zero capability fixture. | no |
| Claude Code | real cache-hit usage counters | yes | Current real-provider probe is blocked by Claude login/auth on EC2. | Authenticate Claude/provider, run repeated identical cacheable prefixes inside TTL, and collect `cache_creation_input_tokens` / `cache_read_input_tokens`. | yes |
| Qwen Code | literal cache key | conditional | Deck frames Qwen cache keying as automatic prefix matching; no literal request cache-key field appeared. | Run against a provider/gateway that exposes cache identity, or add a repeated-prefix provider trace lane. | maybe |
| Qwen Code | real cache-hit usage counters | yes | Local request-boundary capture cannot prove provider cache hits. | Run repeated cacheable Qwen requests against a real provider and collect `usage.cached_tokens` or provider-specific cache counters. | yes |
| Pi Agent Harness | cache-hit feedback | yes | Request-boundary capture saw Pi request fields, but not real provider cache-hit usage. | Run repeated cacheable Pi requests against a real provider and collect Pi footer/provider usage counters such as cache read/write. | yes |
| OpenClaw | provider QoS / service tier | yes | Deck maps fast mode to `service_tier=priority`, but current native capture did not forward `service_tier` or `extra_body.service_tier`. | Identify the OpenClaw provider/config path that forwards service tier to supported OpenAI-compatible requests and rerun `openclaw_provider_qos_fast_mode`. | no |
| OpenClaw | provider cache key | yes | Deck marks cache keying as provider-managed; current request body did not expose `prompt_cache_key`. | Use a provider/path that surfaces prompt-cache request metadata or provider-side cache trace output. | maybe |
| OpenClaw | cache TTL / retention | yes | Provider cache config did not produce a literal retention or TTL field in the local request body. | Exercise a provider path that exposes `prompt_cache_retention` or cache-control TTL metadata. | maybe |
| OpenClaw | cache entry type | yes | Current request body did not expose `cache_control.type`. | Try an OpenClaw provider path with explicit prompt-cache block metadata and inspect message/content blocks. | maybe |
| OpenClaw | cached WebSocket / prewarm metadata | conditional | Deck labels this as a different mechanism from KV prefill; no `cached_websocket` or `websocket_prewarm` field appeared. | Exercise the real cached-WebSocket path and capture WebSocket session metadata or provider trace. | maybe |
| OpenClaw | real cache-hit usage counters | yes | Request-boundary capture cannot prove provider cache hits. | Run repeated cacheable OpenClaw requests against a real provider and collect OpenClaw/provider trace or usage counters. | yes |
| OpenClaw | cache pinning / retention | conditional | Deck says provider-managed only; current capture did not expose retention or literal `cache_pinning`. | Use a provider path that exposes retention metadata, or keep treating this as provider-managed until evidence appears. | maybe |
| Hermes Agent | native request-boundary evidence | yes | Runner/configs are implemented, but no Hermes CLI is installed on this local machine. | Run `--hermes-native-capture` on EC2/GH200 or any host with `HARNESS_HERMES_BIN` set to a working Hermes CLI. | no |
| Hermes Agent | literal cache key | conditional | Deck frames Hermes cache keying as prompt-tier/model/provider behavior; a literal key may not exist. | Run native Hermes repeated-prefix probes and, if needed, a provider trace lane that exposes cache identity. | maybe |
| Hermes Agent | real cache-hit usage counters | yes | Request-boundary capture cannot prove provider cache hits. | Run repeated cacheable Hermes requests against a real provider and collect prompt-cache metrics or usage counters. | yes |
| OpenCode | stats/provider cache feedback | yes | Provider/plugin config capture does not include runtime provider usage. | Run provider/plugin-backed OpenCode requests and collect stats or usage fields that expose cache feedback. | yes |
| Deep Agents | provider trace cache feedback | yes | Fixture recipe exists, but no provider trace run proves cache feedback. | Run repeated cacheable requests and collect provider trace data. | yes |
| DeepSeek Harness | `cacheReadTokens` feedback | yes | Provider-capability capture does not include real provider response counters. | Run repeated cacheable requests and collect `cacheReadTokens` or equivalent usage counters. | yes |
| Codex | cached input-token usage | yes | Provider-config capture does not include real provider/client usage output. | Run repeated cacheable Codex requests and collect cached input-token usage. | yes |

## Missing Or Blocked Today

| Harness | Signal | Current State | Needed To See It |
| --- | --- | --- | --- |
| NAT | cache-hit runtime feedback | Not request-boundary metadata; needs backend/runtime cache metrics. | Run a real backend cache-hit experiment and collect post-execution profiler/metrics output, not only request-boundary capture. |
| NAT | literal `cache_pinning=true` | Not observed in NAT 1.8.0; NAT exposes ephemeral/first-only cache control instead. | A NAT version/path would need to expose an actual pinning field; today use first-only/ephemeral cache control as the observable pin-like behavior. |
| Claude Code | native `service_tier=auto` / `standard_only` body field | Not observed through tested Claude Code env-var path. | A Claude Code/provider path would need to forward <code>ANTHROPIC_SERVICE_TIER</code> or <code>CLAUDE_CODE_SERVICE_TIER</code> into the request body. |
| Claude Code | literal cache key | Not observed; Claude appears to use provider-derived exact-prefix matching. | Claude Code or the provider would need to expose a literal request cache-key field; current native capture did not. |
| Claude Code | tool-level `cache_control` | Not observed in the tested tool-heavy request. | Run a Claude Code setup with stable tool definitions that the CLI/provider marks directly as <code>tools.*.cache_control</code>; our observed path cached system/message blocks instead. |
| Claude Code | native `max_tokens=0` prewarm | Not observed from Claude Code CLI in our capture. | Claude Code would need a native zero-token request path; direct API can represent this, but the native CLI path has not emitted it. |
| Claude Code | real cache-hit usage counters | Implemented runner path, but EC2 Claude CLI is not logged in today. | Use an authenticated Claude/provider run with repeated identical cacheable prefix requests inside the TTL, then capture response usage counters. |
| Qwen Code | literal cache key | Not observed in request-boundary capture; the deck frames Qwen cache keying as automatic prefix matching. | A provider or future Qwen path would need to expose a literal request cache-key field. |
| Qwen Code | real cache-hit usage counters | Request-boundary capture cannot prove provider cache hits. | Use a real provider run with repeated cacheable prefix requests and collect provider usage counters such as cached tokens. |
| Pi Agent Harness | cache-hit feedback | Request-boundary capture saw Pi request fields, but not real provider cache-hit usage. | Use a real provider run with repeated cacheable prefix requests and collect Pi footer/provider usage counters such as cache read/write. |
| OpenClaw | provider QoS / service tier | The deck maps fast mode to `service_tier=priority`, but the current OpenClaw native capture did not forward `service_tier` or `extra_body.service_tier`. | Find the OpenClaw provider/config path that forwards service tier to supported OpenAI-compatible requests, then rerun `openclaw_provider_qos_fast_mode`. |
| OpenClaw | provider cache key, TTL, and entry type | The deck marks these as provider-managed; the current local request-boundary capture did not expose literal cache fields. | Use a provider/path that surfaces prompt-cache request metadata, or collect provider-side cache trace output. |
| OpenClaw | cached WebSocket / prewarm metadata | The deck labels this as a different mechanism from KV prefill; no `cached_websocket` or `websocket_prewarm` request field appeared. | Exercise the real cached-WebSocket path and capture either the WebSocket session metadata or provider trace. |
| OpenClaw | real cache-hit usage counters | Request-boundary capture cannot prove provider cache hits. | Use a real provider run with repeated cacheable prefix requests and collect OpenClaw/provider trace or usage counters. |
| OpenClaw | cache pinning / retention | The deck says provider-managed only; current capture did not expose retention or literal `cache_pinning`. | Use a provider path that exposes retention metadata, or keep treating this as provider-managed until evidence appears. |
| OpenCode | native CLI proof | Provider/plugin config capture exists, but the OpenCode CLI local-provider probe timed out before a request reached the capture server. | Revisit OpenCode native CLI automation with a stricter startup path or prestarted OpenCode server. |
| Deep Agents | native library/CLI proof | Provider/middleware capture exists, but no stable local Deep Agents request-boundary CLI path was proven here. | Add a Deep Agents library-level probe or identify a CLI mode that can redirect model traffic locally. |
| DeepSeek Harness | real provider feedback | Provider-capability capture exists for prefix cache, TTL, and pinning, but `cacheReadTokens` needs a real DeepSeek/provider response. | Run repeated prefix probes against a real provider and collect usage counters. |
| Codex | native Codex client proof | Provider-config capture exists, but native Codex client proof is still separate from provider-config payload evidence. | Run Codex against a local Responses capture server or real provider trace and map the resulting request fields. |
| Hermes Agent | native request-boundary evidence | Runner/configs are implemented, but no Hermes CLI is installed on this local machine. | Run `--hermes-native-capture` on EC2/GH200 or any host with `HARNESS_HERMES_BIN` set to a working Hermes CLI. |
| Hermes Agent | literal cache key | The deck frames Hermes cache keying as prompt-tier/model/provider behavior; a literal cache-key field is optional. | A provider or future Hermes path would need to expose a literal request cache-key field. |
| Hermes Agent | real cache-hit usage counters | Request-boundary capture cannot prove provider cache hits. | Use a real provider run with repeated cacheable prefix requests and collect provider prompt-cache metrics. |

## PPTX Signal Inventory

This table is keyed to `presentation/Harness Signal Tables As-Is.pptx`. The
"possible deck signals" column lists the supported or conditional signals the
deck says each harness may expose. The "observed in this suite" column counts
the strongest evidence lane currently present in this runbook; fixture-only
rows are not counted as observed evidence here.

| Harness | Possible Deck Signals | Observed In This Suite | Still Missing |
| --- | --- | --- | --- |
| Dynamo | soft priority; strict priority tier; expected output length; cache key/bucket; cache namespace/isolation; eviction priority; speculative prefill; cache-hit feedback | 0 / 8 | all deferred; Dynamo intentionally left out for now because of memory/GPU constraints |
| NeMo Agent Toolkit / NAT | soft priority; latency sensitivity; provider speed/QoS tier; expected output length; expected interarrival time; predicted remaining calls; prefix/workflow reuse ID; cache key/bucket; cache namespace/isolation; cache TTL; cache entry type; eviction priority; cache-hit feedback; cache pinning | 12 / 14 | cache-hit runtime feedback; literal cache pinning |
| Claude Code | provider speed/QoS tier; cache key/bucket; cache TTL; cache entry type; cache-hit feedback; cache pinning | 3 / 6 | literal cache key; real cache-hit feedback; native pinning beyond provider TTL retention |
| Codex | provider speed/QoS tier; cache key/bucket; cache namespace/isolation; speculative prefill; cache-hit feedback | 4 / 5 | cached input-token feedback from real provider/client usage |
| DeepSeek Harness | cache key/bucket; cache TTL; cache-hit feedback; cache pinning | 3 / 4 | `cacheReadTokens` or equivalent real-provider feedback |
| Deep Agents | provider speed/QoS tier; cache key/bucket; cache namespace/isolation; cache TTL; cache entry type; eviction priority; cache-hit feedback; cache pinning | 7 / 8 | provider trace cache-hit feedback |
| Qwen Code | provider speed/QoS tier; cache key/bucket; cache namespace/isolation; cache TTL; cache entry type; eviction priority; cache-hit feedback; cache pinning | 5 / 8 | literal cache key; real cache-hit feedback; provider-managed pinning proof |
| Pi Agent Harness | cache key/bucket; cache namespace/isolation; cache TTL; cache entry type; cache-hit feedback; cache pinning | 5 / 6 | real cache-hit feedback |
| OpenCode | cache key/bucket; cache namespace/isolation; cache-hit feedback | 2 / 3 | provider/plugin cache-hit usage feedback |
| OpenClaw | provider speed/QoS tier; cache key/bucket; cache namespace/isolation; cache TTL; cache entry type; speculative prefill; cache-hit feedback; cache pinning | 1 / 8 | all except provider/session namespace |
| Hermes Agent | provider speed/QoS tier; cache key/bucket; cache namespace/isolation; cache TTL; cache entry type; cache-hit feedback; cache pinning | 0 / 7 | native Hermes run still needs a working Hermes CLI; real-provider feedback remains separate |

## Inspect Results

```bash
export RUN_DIR="sglang_direct_kv/artifacts/results/hint_benchmark/<run_id>"
column -s, -t "$RUN_DIR/scenario_validation_summary.csv" | less -S
column -s, -t "$RUN_DIR/hint_support_matrix.csv" | less -S
column -s, -t "$RUN_DIR/unknown_hints.csv" | less -S
```

Key files:

```text
run.json
knob_profile.json
scenario_records.jsonl
observed_hint_evidence.jsonl
expected_hint_evidence.csv
hint_validation.csv
scenario_validation_summary.csv
hint_support_matrix.csv
unknown_hints.csv
```
