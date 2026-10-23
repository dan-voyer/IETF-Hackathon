# AINETOPS Hackathon

Working material for the IETF 127 AINETOPS hackathon, bringing an independently developed Cisco capacity-planning agent into China Telecom's agentic network-management platform.

Keep this repository private. Review partner-shared material with its contributors before publishing it.

## What we are testing

Can an external specialist perform a network-management task and return a result that the operator's platform can interpret and check?

The first experiment is **latency-aware path planning under capacity constraints**. China Telecom's master agent delegates a task over A2A. The specialist retrieves a prepared digital-twin snapshot through MCP, runs an existing planning engine, and returns a candidate path with supporting evidence. China Telecom's validation agent checks the recommendation against the agreed constraints and compatible twin state.

The experiment records **what the implementations still need to agree on beyond A2A and MCP**: task meaning, status mappings, network-data definitions, and evidence. Finding that existing specifications and documented mappings are sufficient is a valid result. No A2A protocol extension is proposed.

## Start here

Read the [hackathon plan](AINETOPS-IETF127-Hackathon.md) for the scenario, component responsibilities, task and evidence models, example messages, test scope, and open questions.

The first event lasts a day and a half. Its core is one successful delegation and validation flow, a “completed but not validated” test, and investigation of any mismatches discovered during integration. Record naturally occurring findings separately from deliberately injected failures.

The specialist has recommendation-only authority. Core validation establishes consistency with the approved fixture and digital-twin model; it does not establish a live-network outcome. Applying a path and measuring its effect in the twin is a stretch goal. Agent lifecycle, intervention, recovery, and live-network changes are outside this first experiment.

## Supporting material

These companion files are planned or awaiting upload. The hackathon plan currently contains the working requirements.

| Purpose | Planned file |
| --- | --- |
| Integration, versioned contract, and data mappings | `Implementation-notes.md` |
| Specialist behavior and evidence requirements | `Capacity-Planning-Agent-Skills-and-Rules.md` |
| Test cases and pass conditions | `Test-plan.md` |
| Agreements, interpretation differences, and resolutions | `Negotiation-log.md` |
| Each team's predictions before integration | `Expected-findings.md` |
| Platform sketches, topology, and lab options | `IETF127-hackathon-proposal.excalidraw` |

The request/result JSON Schema and approved input fixtures with expected results also need to be prepared before integration. Keep credentials out of the repository.

## Public AINETOPS material

The broader problem statement, BoF material, and community discussion are maintained separately in [dan-voyer/IETF/AINETOPS-proposal](https://github.com/dan-voyer/IETF/tree/main/AINETOPS-proposal):

- [Problem statement working text](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/draft-xxx-ainetops-problem-statement-00-v1.md) and [slides](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/AINETOps%20Problem%20Statement.pdf)
- [IETF 127 BoF proposal](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/AINETOPS-BoF-Proposal-IETF127.md) and [BoF request](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/AINETOPS-BoF-Request-IETF127.md)
- [Operator and implementer survey](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/Survey_AI-Enabled_Network_Operations.md)
- [IETF 127 side-meeting proposal](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/AINETOPS-Side-Meeting-IETF127.md), including alignment with ICON

Mailing list: [ainetops@ietf.org](https://mailman3.ietf.org/mailman3/lists/ainetops.ietf.org/) · [Discussion archive](https://mailarchive.ietf.org/arch/browse/ainetops/)
