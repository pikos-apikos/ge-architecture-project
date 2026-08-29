---
name: agent-technical-english
description: Write low-ambiguity execution contracts, HLD requirements, issue resolutions, and agent handoffs for NeoBank Digital Leap using the project's canonical banking terms and authority boundaries.
---

# NeoBank Agent Technical English

Use controlled English when discussion becomes an execution contract. Preserve the approved architecture and make unintended interpretations harder than the intended action.

Agent Technical English (ATE) is inspired by ASD-STE100 Simplified Technical English, but it is not ASD-STE100 and does not claim compliance with that standard. This file adapts ATE to NeoBank Digital Leap.

## Source preservation

When converting source text to ATE:

- preserve meaning, scope, decision owner, constraints, and uncertainty;
- do not add an architectural requirement;
- do not remove a failure or stop condition;
- do not turn an option into a requirement;
- do not guess a vendor, value, identifier, or authority;
- do not change an approved GitHub decision through wording cleanup.

If ambiguity can change the result, mark it `UNKNOWN` and stop the affected action.

## Canonical project language

Read `AGENTS.md` and `docs/glossary.md`. Use one stable term for each concept.

Use these terms exactly when applicable:

- `CBS` — sole financial authority for balances, bookings, and money movement.
- `CBS Transaction Gateway` — the only digital command path to the CBS and the anti-corruption boundary for legacy protocols.
- `ODS` — non-authoritative Enterprise RDBMS read projection.
- `CDC` — source-log change capture that publishes canonical `ACCOUNT`, `BALANCE`, and `TRANSACTION` state changes.
- `recent-write overlay` — CBS-confirmed temporary read-your-writes data that expires when the matching ODS projection is observed.
- `account-scoped quarantine` — ordered isolation of one account's projection stream without stopping healthy accounts.
- `ConsentContext` — short-lived, server-issued verified consent context.
- `Advisor Context Service` — the only approved provider of curated context to the AI Financial Advisor.
- `RC0` through `RC3` — the approved recovery classes.
- `Phase 0` — time-boxed validation and procurement work that must resolve before its dependent delivery gate.

Do not replace these terms with stylistic synonyms.

## Sentence rules

- Write one action per sentence.
- Use imperative sentences for actions.
- Use declarative sentences for facts.
- Put a condition before its dependent action.
- Name the actor and object when either could be confused.
- Prefer `read`, `find`, `compare`, `create`, `update`, `verify`, `record`, `report`, `stop`, and `ask`.
- Replace `handle`, `improve`, `clean up`, `optimize`, or `fix` with observable actions and a completion condition.

## Normative words

Use:

- `MUST` for a required contract rule.
- `MUST NOT` for a prohibited action.
- `MAY` for permitted optional behavior.
- `PREFER` only for a real preference where another choice is allowed and explained.

Requirements in `docs/requirements/requirements-draft.md` MUST use `the system shall ...` and stable FN/NFR identifiers. Do not rewrite those requirement sentences with `MUST`.

## Authority and evidence

For a material decision, write one of:

- `DECISION OWNER: HUMAN`
- `DECISION OWNER: AGENT`
- `DECISION OWNER: EXISTING CONTRACT`

For evidence and uncertainty, use:

- `KNOWN` — supported by an authoritative source.
- `UNKNOWN` — missing information blocks the affected action.
- `ASSUMPTION` — an explicit temporary premise that is safe within the stated scope.

Do not convert `UNKNOWN` to `ASSUMPTION` only to continue.

## NeoBank authority rules

Execution contracts MUST preserve these semantics:

- A transfer is successful only after a canonical CBS `COMMITTED` result.
- A timeout MUST NOT be described as success, rejection, or proof that no booking occurred.
- An unknown CBS outcome remains `PROCESSING` until reconciliation.
- The ODS, cache, report, AI dataset, and recent-write overlay MUST NOT authorize money movement.
- Bank rules decide product eligibility. The AI MAY rank and explain only eligible candidates.
- The AI Financial Advisor MUST NOT receive a direct route to the ODS, DB2, ADABAS, or CBS.
- A quarantined account remains readable from its last complete view with explicit freshness state, but balance-dependent writes stop before Fraud or CBS submission.
- Required transfer transitions, audit receipts, DR promotion evidence, and incident evidence remain immutable, versioned, and reconcilable.

## Execution contract

Use this structure for multi-step or externally visible work:

```text
GOAL: <one observable outcome>
SCOPE: <exact branch, files, issue, environment, or data set>
DECISION OWNER: <HUMAN | AGENT | EXISTING CONTRACT>

1. <action>
2. <action>
3. <verification>

MUST: <hard requirement>
MUST NOT: <hard prohibition>
SIDE EFFECTS AUTHORIZED: <commit | comment | publish | merge | none>
STOP IF: <condition that requires a report or new human decision>
DONE WHEN: <observable state>
```

Do not add fields that do not reduce ambiguity.

## HLD versus LLD

The HLD MAY state an invariant, authority boundary, target, responsibility, or required validation.

The HLD MUST NOT invent exact implementation detail when the approved state leaves it to Phase 0 or LLD. Label such detail as an `LLD carry-forward constraint` and state:

- the invariant the LLD MUST preserve;
- the evidence the LLD MUST produce;
- the value or vendor that remains `UNKNOWN`;
- the gate that resolves the unknown.

## Wayfinder integration

ATE complements Wayfinder. It does not replace the map, ticket types, dependencies, or human gates.

When writing a Wayfinder ticket or resolution:

- write one stable question;
- identify the decision owner;
- keep unresolved decisions unresolved;
- distinguish recommendation from approval;
- quote or closely link the explicit human decision;
- state the affected HLD sections and invariants;
- record completion evidence before closing.

When Wayfinder hands off to execution, compile the handoff into the execution-contract form before any commit, external document update, issue comment, PR update, or merge.

## Side effects

Before a state-changing tool call, verify the exact target, action, identifier, scope, authorization, and done condition.

- Permission to edit does not imply permission to publish.
- Permission to publish does not imply permission to merge.
- A merge instruction applies only to the named PR and verified head SHA.
- If the head SHA changes after verification, stop and re-evaluate the merge.

## Completion check

Before sending an execution contract:

1. Find every action, actor, object, condition, and constraint.
2. Assign each material decision owner.
3. Mark each material unknown.
4. State authorized side effects.
5. Add a stop condition for risky actions.
6. Add an observable `DONE WHEN` condition.
7. Remove vague verbs, pronouns with unclear referents, and unnecessary synonyms.

Use normal language for human discussion. Use ATE only where wording controls execution.

