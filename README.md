# AINETOPS Hackathon

Working material for the IETF 127 AINETOPS hackathon, bringing an independently developed Cisco capacity-planning agent into China Telecom's agentic network-management platform.

Keep this repository private. Review partner-shared material with its contributors before publishing it.

## What we are testing

Can independently developed agents agree on a network-management task, exchange a usable recommendation with evidence, and carry it through to a network outcome?

The experiment uses **bandwidth-aware SR-TE planning**. CT's master delegates to Cisco's CP agent over A2A. The CP agent obtains agreed model data through MCP and returns proposed SR-TE changes with simulation evidence. CT's master checks the recommendation, coordinates application to the lab network, and reports the observed outcome.

The focus is **agent-to-agent interaction**: task meaning, constraints, status, results, evidence, and errors. The two rejection tests—unsupported traffic unit and wrong planning context—run through CT's master. They demonstrate enforcement of the agreed contract. No A2A protocol extension is proposed.

## Start here

- [Demo script](AINETOPS-IETF127-Hackathon.md): operator commands, scenario, rejection tests, master checks, and application to the lab network.
- [Technical reference](AINETOPS-IETF127-Technical-Reference.md): task/status/evidence mappings, illustrative contract, and reporting guidance.

The first event lasts a day and a half. **CT's master performs the validation role at IETF 127**; a separate validation agent is planned for IETF 128. The CP agent recommends changes; CT controls network execution. Simulated utilization, master checks, application status, and observed network behavior are reported separately. Production-network changes are outside scope.

The script identifies remaining implementation checks, including the exact CT application action. Record naturally occurring integration findings separately from injected failures. Existing specifications and documented mappings being sufficient is a valid finding.

## Supporting material

The script and technical reference above are available. The following preparation records remain planned or awaiting upload:

| Purpose | Planned file |
| --- | --- |
| Integration, versioned contract, and data mappings | `Implementation-notes.md` |
| Specialist behavior and evidence requirements | `Capacity-Planning-Agent-Skills-and-Rules.md` |
| Test cases and pass conditions | `Test-plan.md` |
| Agreements, interpretation differences, and resolutions | `Negotiation-log.md` |
| Each team's predictions before integration | `Expected-findings.md` |
| Platform sketches, topology, and lab options | `IETF127-hackathon-proposal.excalidraw` |

Keep the implemented request/result schema, fixtures, and captured exchanges with the preparation records when available. Keep credentials out of the repository.

## Public AINETOPS material

The broader problem statement, BoF material, and community discussion are maintained separately in [dan-voyer/IETF/AINETOPS-proposal](https://github.com/dan-voyer/IETF/tree/main/AINETOPS-proposal):

- [Problem statement working text](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/draft-xxx-ainetops-problem-statement-00-v2.md) and [slides](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/AINETOps%20Problem%20Statement.pdf)
- [IETF 127 BoF proposal](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/AINETOPS-BoF-Proposal-IETF127.md) and [BoF request](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/AINETOPS-BoF-Request-IETF127.md)
- [Operator and implementer survey](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/Survey_AI-Enabled_Network_Operations.md)
- [IETF 127 side-meeting proposal](https://github.com/dan-voyer/IETF/blob/main/AINETOPS-proposal/AINETOPS-Side-Meeting-IETF127.md), including alignment with ICON

Mailing list: [ainetops@ietf.org](https://mailman3.ietf.org/mailman3/lists/ainetops.ietf.org/) · [Discussion archive](https://mailarchive.ietf.org/arch/browse/ainetops/)
