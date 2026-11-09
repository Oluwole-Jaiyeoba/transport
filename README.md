# Proactive KV Management: Code and Integration Map

This document is the code-level reference for
`controller_proactive_kv_management`. It shows how the mode uses an expected
tool-return time to decide whether and when to restore a session's KV prefix
from host memory to GPU memory.

Use this document before moving the mode into another testbed. It distinguishes
controller intent, admission policy, backend dispatch, native SGLang behavior,
and experiment-only configuration.

## Short definition

The mode follows this path:

```text
tool wait begins
    -> calculate expected replay time
    -> estimate KV load time
    -> reserve a safety margin
    -> choose the latest safe load start
    -> ask SGLang whether useful host-resident KV exists
    -> load eligible KV through SGLang's native HiCache path
    -> record ready, skipped, late, unsupported, or failed
```

The expected replay time is the expected tool-return time in these experiments:

```python
replay_due_ms = tool_start_ms + wait_ms
```

It is an estimated return deadline, not a guarantee that an external tool will
finish at precisely that time.

## End-to-end code evidence

| Claim | What the code does | Code reference |
|---|---|---|
| The mode exists | Defines `controller_proactive_kv_management`. | [`modes.py`](../../../packages/agentic-controller/src/agentic_controller/modes.py#L13) |
| It is supported by the driver | Includes the mode in `SUPPORTED_MODES`. | [`modes.py`](../../../packages/agentic-controller/src/agentic_controller/modes.py#L65-L79) |
| It uses targeted KV prefetch underneath | The targeted-prefetch family explicitly includes proactive management. | [`modes.py`](../../../packages/agentic-controller/src/agentic_controller/modes.py#L134-L140) |
| Proactive behavior is separately detectable | A dedicated predicate distinguishes proactive mode from basic targeted prefetch. | [`modes.py`](../../../packages/agentic-controller/src/agentic_controller/modes.py#L143-L148) |
| Tool return becomes a replay-due time | Every tool wait calculates `replay_due_ms = tool_start_ms + wait_ms`. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L4264-L4277) |
| The controller receives that expected return time | `TOOL_STARTED` is recorded with `expected_completion_ms=replay_due_ms`. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L4308-L4321) |
| Harness timing is normalized | The portable signal records tool start, estimated duration, expected completion, uncertainty, wait class, and wait duration. | [`signals.py`](../../../packages/agentic-harnesses/src/agentic_harnesses/signals.py#L211-L239), [`signals.py`](../../../packages/agentic-harnesses/src/agentic_harnesses/signals.py#L278-L299) |
| The controller reasons about time remaining | It calculates `expected_completion_ms - now_ms` and decides whether the session is in its prepare window. | [`policy.py`](../../../packages/agentic-controller/src/agentic_controller/policy.py#L86-L112) |
| The controller emits a prefetch command | Inside the prepare window, it creates `KVAction.PREFETCH` with reason `restore KV before replay`. | [`policy.py`](../../../packages/agentic-controller/src/agentic_controller/policy.py#L200-L228) |
| Loading time is estimated | The estimate is `(fixed_ms + prompt_tokens * ms_per_token) * multiplier`. | [`lead_times.py`](../../../packages/agentic-controller/src/agentic_controller/lead_times.py#L50-L55) |
| A safety margin is reserved | The latest acceptable finish is `replay_due_ms - safety_margin_ms`. | [`lead_times.py`](../../../packages/agentic-controller/src/agentic_controller/lead_times.py#L72-L77), [`lead_times.py`](../../../packages/agentic-controller/src/agentic_controller/lead_times.py#L99-L105) |
| Wait classes can be restricted | The controller can allow only selected classes, such as `long`. | [`lead_times.py`](../../../packages/agentic-controller/src/agentic_controller/lead_times.py#L58-L63), [`lead_times.py`](../../../packages/agentic-controller/src/agentic_controller/lead_times.py#L106-L117) |
| Unsafe windows are rejected | It skips when no safe window exists before replay. | [`lead_times.py`](../../../packages/agentic-controller/src/agentic_controller/lead_times.py#L118-L129) |
| Loads estimated to finish late are rejected | It compares estimated loading time with available slack. | [`lead_times.py`](../../../packages/agentic-controller/src/agentic_controller/lead_times.py#L130-L141) |
| The latest safe start is selected | It starts no earlier than necessary while retaining estimated load time and safety margin. | [`lead_times.py`](../../../packages/agentic-controller/src/agentic_controller/lead_times.py#L142-L156) |
| The driver applies the timing calculation | It passes tool start, replay due time, prompt tokens, and wait class into the load-window policy. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L4392-L4404) |
| The load waits until the chosen start | The asynchronous task calls `sleep_until(targeted_prefetch_start_ms)`. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L4495-L4508) |
| Expired work is abandoned | The controller records `direct_load_skipped_expired` when the safe finish window has passed. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L4497-L4508) |
| The controller checks usefulness before loading | It sends a `plan_only=True` request before the real load. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L4509-L4515) |
| Small host prefixes are skipped | Host-resident KV below the configured token threshold is rejected. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L3558-L3566) |
| Useful host prefixes are admitted | A sufficiently large host-resident, evicted prefix is admitted. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L3567-L3573) |
| GPU-resident prefixes are skipped | `already_device_resident` is treated as no load needed. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L3574-L3581) |
| The controller calls a direct endpoint | It posts session, prefix, replay identity, and deadline-derived timeouts to `/prepare_prefix_kv`. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L3466-L3511) |
| Fake model warmups are not the direct mechanism | Only `prepared_prefix_control` is authorized; the legacy synthetic-request mechanism is rejected. | [`lead_times.py`](../../../packages/agentic-controller/src/agentic_controller/lead_times.py#L80-L89) |
| The backend exposes the endpoint | The SGLang adapter accepts POST requests at `/prepare_prefix_kv`. | [`patch.py`](../../../packages/agentic-backend-sglang/src/agentic_backends/sglang/trace/patch.py#L2374-L2408) |
| The backend selects real host-resident KV | A candidate must be evicted, host-backed, and large enough. | [`patch.py`](../../../packages/agentic-backend-sglang/src/agentic_backends/sglang/trace/patch.py#L2044-L2069) |
| The backend performs native SGLang loading | It calls `tree_cache.load_back(node, mem_quota)`. | [`patch.py`](../../../packages/agentic-backend-sglang/src/agentic_backends/sglang/trace/patch.py#L2141-L2167) |
| HiCache launches the host-to-device work | It calls `tree_cache.ready_to_load_host_cache()` on SGLang's load stream. | [`patch.py`](../../../packages/agentic-backend-sglang/src/agentic_backends/sglang/trace/patch.py#L2205-L2224) |
| Native completion can be observed | The endpoint waits for the prepared load and reports `ready` only after native completion is observed. | [`patch.py`](../../../packages/agentic-backend-sglang/src/agentic_backends/sglang/trace/patch.py#L2270-L2294) |
| Outcomes are recorded | The driver records successful readiness, errors, and timeouts. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L4544-L4593) |
| Peer/filler sessions are included | Proactive mode gives filler-session tool waits the same timing-aware callback. | [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L1623-L1671), [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L4016-L4030) |

## Reference experiment configuration

The maintained Scenario 2 specification isolates early KV preparation from
queue scheduling and eviction policy.

| Setting | Reference behavior | Code reference |
|---|---|---|
| Mode | `controller_proactive_kv_management` | [`controller_audit_pivots.json`](../../../configs/experiment_specs/controller_audit_pivots.json#L60-L64) |
| Equal application importance | Semantic high/low priority inputs are disabled for this reference experiment. | [`controller_audit_pivots.json`](../../../configs/experiment_specs/controller_audit_pivots.json#L8-L23), [`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L420-L452) |
| Tool waits | Long synthetic waits of 60–120 seconds create visible preparation opportunities. | [`controller_audit_pivots.json`](../../../configs/experiment_specs/controller_audit_pivots.json#L65-L67) |
| Direct hook | `AGENTIC_KV_TARGETED_PREFETCH_HOOK=1` | [`controller_audit_pivots.json`](../../../configs/experiment_specs/controller_audit_pivots.json#L67) |
| Control endpoint | `/prepare_prefix_kv` is enabled on port 31991. | [`controller_audit_pivots.json`](../../../configs/experiment_specs/controller_audit_pivots.json#L68-L72) |
| Load estimate | Fixed 1,000 ms plus 2 ms per token, multiplied by 3. | [`controller_audit_pivots.json`](../../../configs/experiment_specs/controller_audit_pivots.json#L73-L76) |
| Safety margin | The planned finish is at least 750 ms before replay due time. | [`controller_audit_pivots.json`](../../../configs/experiment_specs/controller_audit_pivots.json#L77) |
| Completion contract | The reference run requests completed direct loading rather than best effort. | [`controller_audit_pivots.json`](../../../configs/experiment_specs/controller_audit_pivots.json#L78) |
| Selective admission | Only long waits and prefixes saving at least 1,024 tokens qualify. | [`controller_audit_pivots.json`](../../../configs/experiment_specs/controller_audit_pivots.json#L79-L81) |

The runnable comparison is:

```bash
bash sglang_direct_kv/scripts/run_proactive_kv_management_realistic.sh \
  Qwen/Qwen2.5-Coder-7B-Instruct
```

The script compares `no_prefetch` with
`controller_proactive_kv_management`: [`run_proactive_kv_management_realistic.sh`](../../../sglang_direct_kv/scripts/run_proactive_kv_management_realistic.sh#L13-L49).

## Existing verification tests

| Test | What it proves | Code reference |
|---|---|---|
| Direct-load window rejection | A load is rejected when replay is too close. | [`test_lead_times.py`](../../../packages/agentic-controller/tests/test_lead_times.py#L21-L24) |
| Missing backend hook | The adapter reports the action as unsupported instead of claiming success. | [`test_lowering_adapters_telemetry_launch.py`](../../../packages/agentic-backend-sglang/tests/test_lowering_adapters_telemetry_launch.py#L68-L72) |
| Available backend hook | The adapter marks the command as dispatched to the backend. | [`test_lowering_adapters_telemetry_launch.py`](../../../packages/agentic-backend-sglang/tests/test_lowering_adapters_telemetry_launch.py#L73-L74) |

## Important integration qualifications

### 1. The tool ETA must come from somewhere trustworthy

In the synthetic testbed, the expected return time comes from the configured
wait duration. A real harness integration must supply an expected completion
time or estimated tool duration. SGLang does not discover the tool-return time
on its own.

### 2. Equal importance belongs to the experiment, not the mode itself

The maintained Scenario 2 experiment enables
`AGENTIC_EQUAL_IMPORTANCE_WORKLOAD=1`. The proactive mode does not automatically
erase semantic priorities in every possible caller. A new testbed must enable
the equal-importance contract if that is part of the experiment design.

### 3. `REQUIRE_COMPLETION` is not an indefinite replay gate

The direct request waits for native readiness within its deadline-derived
timeout. However, replay still becomes eligible at its due time. A late or
failed preparation is recorded rather than allowing the controller to block
replay indefinitely. The replay is submitted at
[`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L4872-L4966)
and executed at
[`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L5135),
while the preparation task is collected afterward at
[`run_multi_harness_replay_driver.py`](../../../packages/agentic-experiments/src/agentic_experiments/runners/run_multi_harness_replay_driver.py#L5179-L5180).

### 4. Backend support must be proved, not assumed

The adapter only marks a command as acted when the direct hook is available.
Without it, the result is explicitly `UNSUPPORTED`:
[`adapters.py`](../../../packages/agentic-backend-sglang/src/agentic_backends/sglang/adapters.py#L330-L391).

### 5. Static code proves policy, while traces prove a particular run

The code establishes what the controller is designed to attempt. A run should
only be called successful when its trace links:

1. tool-wait start and expected replay identity;
2. admission or skip decision;
3. prepare-plan result;
4. native SGLang load result;
5. load completion relative to replay due time; and
6. the matching replay request.

The historical reference records 21 preparation requests reaching the backend,
with nine requiring load plans and twelve already resident:
[`controller_experiment_registry.json`](../../../configs/controller_experiment_registry.json#L102-L127).

## Minimum porting checklist

Another testbed should not claim this mode is active until it can answer yes to
all of these questions:

- Does every session have a stable `session_id`, `prefix_id`, and replay ID?
- Does the harness expose tool start plus expected completion or duration?
- Is `replay_due_ms` represented in one consistent monotonic clock domain?
- Is the direct-load estimate calibrated for the target model and machine?
- Does the backend provide an honest plan-only residency check?
- Does the adapter distinguish unsupported, skipped, dispatched, ready, and failed?
- Does the backend operation invoke native KV movement rather than a fake model request?
- Can traces join the preparation attempt to the exact later replay?
- Does the experiment preserve equal importance if that is part of its claim?
- Does a missing required hook fail loudly before measurements are trusted?

## Precise behavioral statement

> Proactive KV management uses the expected tool-return time as a preparation
> deadline, estimates whether direct host-to-GPU KV restoration can finish
> safely before that deadline, starts it at the latest safe point, and skips
> operations that are late, unnecessary, unsupported, or too small to justify.
