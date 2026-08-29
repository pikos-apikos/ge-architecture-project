---
name: wayfinder
description: Navigate NeoBank Digital Leap architecture work through one auditable GitHub decision ticket at a time while preserving the approved HLD, explicit human gates, and repository source-of-truth rules.
---

# NeoBank Wayfinder

Maintain the decision route for the NeoBank Digital Leap HLD and its governed follow-on work. Use GitHub issues as the durable audit trail. Resolve uncertainty before execution and preserve the human's authority over material architecture choices.

This project variant is adapted from Matt Pocock's MIT-licensed [Wayfinder](https://github.com/mattpocock/skills/blob/main/skills/engineering/wayfinder/SKILL.md) and the OpenAI Wayfinder variant.

## Project sources of truth

Read `AGENTS.md`, `project-status.md`, the active Wayfinder map, and the selected ticket before acting.

Apply this precedence:

1. The course brief and HLD template define submission constraints.
2. Approved GitHub issue decisions define human-approved architecture choices.
3. Mermaid sources define diagram topology when prose and a diagram differ.
4. Canonical Markdown under `docs/` defines the authoring source.
5. The generated Word or Google Docs artifact is a rendered deliverable. It MUST NOT silently replace the canonical repository decision record.

Preserve all architectural invariants in `AGENTS.md`. If a requested change conflicts with an invariant or an approved ticket, stop and surface the conflict.

## Map contract

Use one issue labelled `wayfinder:map` as the low-resolution index. The NeoBank map is [NeoBank Digital Leap Wayfinder Map](https://github.com/pikos-apikos/ge-architecture-project/issues/2).

The map MUST contain:

- `Destination`
- `Notes`, including the execution boundary
- linked one-line entries under `Decisions so far`
- `Not yet specified`
- `Out of scope`

Store detailed analysis, alternatives, approvals, and evidence on the decision ticket. Do not duplicate the full decision in the map.

## Ticket contract

Create only a precise question that can be resolved in one focused conversation or bounded investigation.

```text
QUESTION: <one material question>
WHY NOW: <the later choice this answer unblocks>
DECISION OWNER: <HUMAN | AGENT | NONE>
OUTPUT: <alternatives | recommendation | evidence | completed prerequisite>
SELECTION REQUIRED BEFORE CLOSE: <YES | NO>
AGENT MAY SELECT: <YES | NO>
PARALLEL EXECUTION: <PROHIBITED | REQUIRES EXPLICIT APPROVAL | APPROVED>
DONE WHEN: <observable ticket resolution>
```

Apply one label: `wayfinder:grilling`, `wayfinder:prototype`, `wayfinder:research`, or `wayfinder:task`.

## Human gate

- Ask one material question at a time.
- Offer materially distinct alternatives when a choice is required.
- A recommendation MAY identify the preferred option and its trade-offs.
- The agent MUST NOT treat its recommendation as the decision.
- The agent MUST NOT treat `continue`, `next`, `go on`, or similar language as approval of an unnamed option.
- Close a human-gated ticket only after an explicit selection, rejection, deferral, or delegation.
- Record the user's actual decision closely enough for a later reviewer to audit the gate.

For the Final HLD, changing an approved authority boundary, consistency invariant, security boundary, recovery class, contract semantic, delivery baseline, or merge state requires explicit human approval.

## Work one frontier

1. Load the map at low resolution.
2. Select the user-named ticket, or recommend exactly one unblocked frontier ticket.
3. Claim the ticket when assignment is available.
4. Load only the evidence required for that ticket.
5. Resolve it according to its label and human gate.
6. Write the durable resolution comment.
7. Close the ticket only when its completion contract is satisfied.
8. Add one linked gist to the map.
9. Re-evaluate visible dependencies and fog.
10. Recommend one next ticket. Do not start it in the same decision cycle.

Independent research MAY run in parallel only after explicit authorization. Human decision tickets MUST NOT run in parallel.

## Final-HLD execution handoff

When the map authorizes implementation or document assembly, compile the handoff with Agent Technical English.

The handoff MUST identify:

- the approved issue decisions that govern the change;
- the exact repository branch and files that MAY change;
- whether the Google Docs or Word artifact must also be updated;
- the required diagram, table, traceability, and render checks;
- whether external comments, commits, PR updates, or merge are authorized;
- the observable completion condition.

Publishing a commit does not authorize merging. An explicit merge instruction authorizes the merge only for the resolved PR and current verified head SHA.

## Post-lock corrections

A post-lock editorial or carry-forward correction MAY be applied without reopening the architecture map only when it does not change an approved architecture decision.

The agent MUST:

1. update the canonical source;
2. update the rendered deliverable when it is in scope;
3. record the change and verification evidence on the final-lock ticket;
4. identify any new LLD obligation explicitly;
5. stop if the correction changes a material boundary or baseline.

## Failure checks

Stop and correct course if:

- the map becomes a fixed implementation backlog;
- a ticket hides multiple material decisions;
- research silently selects a product or architecture choice;
- a generated document contradicts the approved repository sources;
- a generic continuation message is recorded as approval;
- two live human gates overlap;
- an issue closes because an artifact exists but the human gate is unsatisfied;
- work expands beyond the selected ticket or explicit execution handoff.

