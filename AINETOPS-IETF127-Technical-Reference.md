# AINETOPS IETF 127 technical reference

[Return to the hackathon plan](AINETOPS-IETF127-Hackathon.md).

This companion describes the agent-to-agent contract and evidence. The main script covers the demo, CT master checks, and application to the lab network through CT's network-control function. A separate validation agent is planned for IETF 128. Existing MCP input specifications are not repeated here.

## Technical reference: task and evidence model

The SR-TE scenario makes the agent exchange concrete: what was requested, which constraints crossed the boundary, what the CP agent returned, and what CT checked and applied. Use these mappings to document the implemented contract and retain a [negotiation log](#the-negotiation-log-is-the-primary-evidence).

CT's existing model (slide *Task Model / Evidence Model in Configuration Generation*) has these task fields: `Task_id`, `Type`, `Goal`, `Target`, `Mode` (for example `PLAN (READ_ONLY)`), `Window`, `Current-step`, `Current-Job`, `Process_flow`, `Status`, `Exist_reason`, `Check_point`, `Device`, `Process_result`, `Output`. Its evidence is `dict(step, JSON)`, used today only in the validation stage. CT should confirm whether these fields are what its platform implements or an illustration for the configuration-generation use case. The spelling `Exist_reason` is retained from the supplied model; confirm whether the implemented field is `Exist_reason` or `Exit_reason` before finalizing the mapping.

### Carry a versioned network contract in A2A

Use a versioned planning request/result contract carried in standard A2A 1.0 message data parts and result artifacts. This hackathon does not define or negotiate an A2A protocol extension. Agree the application contract identifier, supported versions, required fields, and unsupported-version behavior before testing. The JSON examples below identify that contract through `contract: "bandwidth-planning/v1"` inside the application data.

Retain the implemented request/result schema, fixtures, and expected error outcomes. The script's `prompt` and `constraints` objects are **CT chatbot inputs**; capture the forwarded CP-agent request to document their actual mapping. The `bandwidth-planning/v1` example below is a **proposed contract**, not a captured exchange. Preserve the test constraints across the handoff. Schema validation alone does not establish shared meaning.

The useful finding is which network semantics required an additional agreement, whether existing models already define them, and whether the issue is network-specific or generic. For each field below, distinguish the core A2A carrier from our application-defined meaning. Document that meaning in the request/result contract and record any remaining out-of-band agreements.

### Task model

| What the task must say | CT field | Where A2A 1.0 carries it | What must be agreed |
| --- | --- | --- | --- |
| Identity | `Task_id` | Core: task ID, context ID | Map CT’s ID to the specialist’s server-assigned A2A task ID; evidence cites both. Context ID groups interactions and is not a task ID |
| Kind of task | `Type` | Agent Card advertises skills; request data identifies our operation | Skill and operation mapping; the card alone does not select a per-request operation |
| Objective | `Goal` | Core: message text or structured data part; objective semantics are application-defined | Reduce maximum interface utilization toward 70% for the unchanged 1,200 Mbps demand; agree units, direction, and model revision |
| Network scope | `Target`, `Device` | Application contract in a data part; content not defined by A2A | How nodes, links, and endpoints are named on both sides, for example [RFC 8345](https://www.rfc-editor.org/info/rfc8345) IDs |
| Constraints | Inside `Goal`? | Application contract in a data part | Preserve the 1,200 Mbps demand, 1,000 Mbps interface capacities, and reachability; target maximum utilization of 70% |
| Authority | `Mode` | Application contract; platform/tool access controls enforce it | CP agent edits the planning model; CT controls application to the lab network. Production-network changes are excluded |
| Time validity | `Window` | Application contract in a data part | Task validity or data measurement window? |
| Input data | Not present | Application contract in a data part | Which snapshot to use |
| Progress | `Process_flow`, `Current-step`, `Check_point` | Status message, history | Whether a sub-agent reports steps, and whether CT's human check points apply to it |
| Why it ended | `Exist_reason` | Status message | Reasons that separate "no feasible path" from bad input or engine error |
| Output | `Process_result`, `Output` | Artifacts | Result structure, kept separate from evidence |

### Task status

CT's status values and A2A's task states do not line up one to one. A2A 1.0 explicitly rejects `TASK_STATE_RUNNING`; its equivalent is `TASK_STATE_WORKING`.

| CT status | Closest A2A 1.0 state | What must be agreed |
| --- | --- | --- |
| `RUNNING` | `WORKING` | A rename, but the names are not shared |
| `WAITING` | `INPUT_REQUIRED`, `AUTH_REQUIRED`, or `SUBMITTED` | Waiting for whom? |
| `END` | `COMPLETED` | Specialist computation completed, or CT master checks completed? |
| `ERR` | `FAILED` | Is "no feasible path" an error or a completed negative answer? |
| `EXIT` | `CANCELED`? | How it differs from `END` and `ERR` |
| Record CT mapping | `REJECTED` | How CT presents a refused task and preserves the specialist's rejection reason |

### Outcome layers

For this planning contract, the CP agent's `TASK_STATE_COMPLETED` means it finished the requested analysis, including a legitimate no-feasible-path answer. This is our task-specific completion criterion, not A2A's universal definition. The master reports the highest outcome layer actually reached and never infers a higher one. State names abbreviated in the tables use the `TASK_STATE_` prefix on the A2A 1.0 JSON wire.

A completed, failed, canceled, or rejected A2A task is terminal. Correction or replanning after completion creates a new task, linked to the prior task using `referenceTaskIds` and, where appropriate, the same `contextId`. Do not send a correction as another message on the completed task. The overall CT workflow may remain open while the specialist's individual task is complete.

The unit test reports `input_invalid`; the plan-context test reports `TASK_STATE_REJECTED` with `constraint_rejected`. Preserve that distinction in CT's response and record where rejection occurred, with a task ID when one exists.

| Layer | Reported by | In this event |
| --- | --- | --- |
| 1. Accepted | CP agent: `SUBMITTED` or `WORKING` | Core |
| 2. Computed | CP agent: `COMPLETED` with result and evidence | Core |
| 3. Master-checked | CT master: agreed required checks PASS with matching evidence; specialist completion alone is insufficient | Core at IETF 127; separate-agent validation planned for IETF 128 |
| 4. Applied to the lab network | CT network-control function: execution status and configuration read-back | Core; planning-model edits do not establish application |
| 5. Observed network outcome | CT platform: fresh lab-network observations | Core; compare measured results with the target; re-simulation alone is insufficient |

### Evidence model

The fields combine R-6 (provenance, confidence, trace context, verification) with the v2 deck (source, time, scope, operation reference, verification result). The specialist produces planning evidence, and CT's master records its own checks for the operator. There is no separate validation agent at IETF 127.

| Field | CP agent's evidence | CT master's check record |
| --- | --- | --- |
| Task reference | A2A task ID and CT `Task_id` | Same IDs, plus the master's check step |
| Source | CP agent's authenticated identity | CT master's identity |
| Time | Snapshot collection times, run time | Check time and model-state time |
| Scope | Demand A_to_D, A to D; Tunnel100 and Tunnel200 | Same demand, affected LSPs and interfaces |
| Inputs | Snapshot IDs retrieved over MCP | Compatible twin state ID, same proposed SR-TE changes, demand, capacities, and constraints; explain any state differences |
| Operation reference | Engine operation, parameters, run ID | Master check ID and references to inspected simulation reports |
| Result | Actual baseline and optimized simulation results (recorded run: 120% → 60%), unchanged demand/capacity, and SR-TE changes | Checks of reported utilization, demand, capacity and available reachability evidence; identify checks not performed |
| Verification | Engine-backed prediction with traceable output; not independently validated | PASS, FAIL, or INCONCLUSIVE, with method and checks performed |
| Limitations | Model prediction and assumptions | Master checks; no claim of independent recomputation; record shared assumptions and unchecked claims |
| Access | Reference to full engine output, and who may read it | Accessible check record and references to inspected evidence |

Record how evidence travels—in an A2A artifact or accessible reference—and how CT correlates it with the task. CT separately retains execution status, configuration read-back, and post-change observations. A structured record with saved logs is enough; simulation evidence alone does not establish network application.

### Illustrative A2A planning request

This illustrative A2A 1.0 HTTP+JSON request uses `POST /message:send`, `Content-Type: application/a2a+json`, and `A2A-Version: 1.0`. It is not captured traffic or a complete schema. Fields inside `data` belong to the proposed planning contract. The authority restriction applies to the CP agent; CT's later network execution is separate. Record the actual binding and contract used in the demo.

The first request omits `taskId`: the specialist assigns its A2A task ID. `ctTaskId` preserves the master's correlation ID.

```json
{
  "message": {
    "messageId": "msg-plan-001",
    "role": "ROLE_USER",
    "parts": [{"data": {
      "contract": "bandwidth-planning/v1",
      "ctTaskId": "CT-001",
      "operation": "optimize-sr-te",
      "demand": {"name": "A_to_D", "source": "A", "destination": "D", "traffic": 1200, "unit": "Mbps"},
      "objective": {"metric": "maximum-interface-utilization", "target": 70, "unit": "percent"},
      "constraints": {"preserveDemand": true, "preserveCapacities": true, "preserveReachability": true},
      "input": {"modelRevision": "lab-overloaded-001"},
      "authority": {"planningModelCopyChanges": true, "liveNetworkChanges": false},
      "requiredEvidence": ["baseline-simulation", "sr-te-change-set", "sr-te-change-report", "optimized-simulation"]
    }}]
  }
}
```

### Result artifacts and the proposed SR-TE changes

Return a result artifact with task identifiers, planning outcome, requested target, and baseline/optimized utilization, plus an accessible **SR-TE change artifact**. CT maintains its check and execution status separately. A count such as “two LSPs rerouted” is a summary, not the proposed plan.

| Change-artifact content | Required information |
| --- | --- |
| Binding | Contract version, CT/A2A task IDs, engine run ID, exact input and output model revisions |
| LSP changes | Stable LSP ID, source/destination, and the original and proposed SR-TE configuration for each changed LSP |
| Segments and weights | Ordered segment lists, segment identifiers/types, and weights, including their meaning and normalization; readable node names alone are insufficient where the engine uses other identifiers |
| Forwarding interpretation | Expanded routes or accessible engine details explaining how the segment lists resolve in the referenced model |
| Demand binding | How A_to_D is mapped to the changed LSPs, with the demand amount and units preserved |
| Interface impact | Identified affected forward interfaces, unchanged capacities, and simulated utilization before and after optimization |
| Supporting evidence | Accessible baseline/optimized reports and original engine change output, linked to the same run and model revisions |

Carry structured changes directly in an A2A artifact, or provide an agreed accessible reference to the complete change artifact. The master must be able to retrieve and interpret it using the agreed format, without a separate explanation from an engineer. Pin that application format and version; it is not an A2A protocol extension. Do not put credentials in artifact references.

The local display provides summaries for Tunnel100 and Tunnel200, reproduced in the [main scenario](AINETOPS-IETF127-Hackathon.md#the-scenario-from-an-overloaded-path-to-a-balanced-sr-te-plan). Those summaries are not complete configurations. Populate the change artifact from the engine's actual output; do not reconstruct omitted segment identifiers, routes, weights, or demand assignments from the screenshots or summary percentages.

The result may summarize changes and metrics, but references must resolve to the corresponding accessible content. Example identifiers are not substitutes for retained artifacts. An unmet target can still be a completed analysis, with an explicit outcome and explanation. The specialist cannot assign the master's check outcome.

CT's master adds its own check record: the methods, evidence inspected, check outcomes, and unchecked claims. See the [minimum checks](AINETOPS-IETF127-Hackathon.md#minimum-checks-by-cts-master). This is the temporary IETF 127 validation role, not independent recomputation. A correction after a terminal specialist task creates a new linked task, using `referenceTaskIds` and, where appropriate, the same `contextId`.

## Making the findings credible

### The negotiation log is the primary evidence

Record what CT and Cisco had to agree on: each interpretation, operational consequence, existing mechanism considered, and resolution. Note whether another implementation could reuse that resolution. Integration effort alone does not establish a standards gap; the demonstrated exchange and retained evidence must support the conclusion.

### Distinguish integration findings from injected tests

The [two rejection tests](AINETOPS-IETF127-Hackathon.md#6-check-that-incompatible-requests-are-rejected)—unsupported traffic unit and wrong planning context—demonstrate contract enforcement through CT's master. They do not establish an industry-wide gap or validate returned evidence. Tag findings as **organic** or **injected**, and record where existing mechanisms were sufficient.

### Keep predictions and observations separate

Preserve any predictions made before testing with their original dates. Distinguish them from explanations written after seeing the results. Record unexpected findings and predictions that did not hold.

### Optional independent review

An outside reviewer or separately implemented contract consumer can strengthen the findings. This is additional work, not required for the core demo. A generic A2A SDK client alone does not establish portability of the network-management contract.

## Production freshness and pre-execution checks

For operational bandwidth-aware path selection, inputs must be fresh enough for the decision, typically using near-real-time topology, availability, link capacity, utilization, and service traffic from the operator platform. The specialist can consume a consistent snapshot when tasked rather than maintain its own telemetry feed. Agree maximum input age, measurement windows, source alignment, validity conditions, and changes that invalidate a result. Long-term capacity forecasting also needs representative history and growth assumptions. The demo uses prepared planning inputs and post-change lab observations; continuous production telemetry remains outside scope.

Before applying an approved plan, a separate authorized executor checks current conditions and permissions. If the plan would now cause congestion or violate constraints, pause and request replanning through a linked new task. If another path is merely better while the approved path remains valid, operator policy determines whether to proceed or optimize again; use thresholds and a replanning budget to avoid oscillation. Materially revised plans require authorization as appropriate. Current-state checks alone do not prevent concurrent resource changes; controller-supported admission or reservation mechanisms may be needed. After execution, assurance measures the outcome and coordinates authorized recovery when necessary; rollback is not automatically safe.

## Background references

- [AINETOPS problem statement (draft)](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/draft-xxx-ainetops-problem-statement-00-v2.md) and [slides (PDF)](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/AINETOps%20Problem%20Statement.pdf)
- [AINETOPS BoF proposal for IETF 127](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/AINETOPS-BoF-Proposal-IETF127.md)
- [A2A specification](https://a2a-protocol.org/latest/specification/). Record the actual version used in the experiment.
- [MCP tool specification, 2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28/server/tools)
- [YANG 1.1, RFC 7950](https://www.rfc-editor.org/info/rfc7950/), [network topology, RFC 8345](https://www.rfc-editor.org/info/rfc8345), [TE topology, RFC 8795](https://www.rfc-editor.org/info/rfc8795)
- [A2A device-agent applicability draft](https://datatracker.ietf.org/doc/html/draft-yan-a2a-device-agent-applicability-02). An individual proposal, not an adopted requirement.
- [AINETOPS discussion: MCP interfaces, semantics, and initial OPS guidance](https://mailarchive.ietf.org/arch/msg/ainetops/2OwThqqge-X5zGRo5HVizWdcPBo/)
- [Brad Peters: limitations in extracting operational meaning from YANG](https://mailarchive.ietf.org/arch/msg/ainetops/q0xy2o1QZ9H2kgEm2nuPSfoFOlg/)
- [Discussion separating AI-to-model interpretation and model mapping](https://mailarchive.ietf.org/arch/msg/ainetops/qLUWWTDYrqL3g8z0e813kOulT-8/)
- [Discussion of approaches beyond YANG and the IETF/IRTF boundary](https://mailarchive.ietf.org/arch/msg/ainetops/Z2MdGTdZ51ajDB48RSihB8gGzh4/)

The cited mailing-list exchanges express participant views, not community consensus.
