# Domain demo specification template

Copy this file to the relevant domain folder and complete every bracketed
instruction before implementation. Keep the design scoped to one independently
deployable solution.

## Use case and narrative

- **Domain and solution:** `<domain>/<demo>/<demo>-solution/`
- **Enterprise use case:** [Describe the event, user role, business decision,
  and the visible hero moment.]
- **Audience journey:** [State what the demonstrator shows from intake through
  completion, and why Flow is the right orchestration surface.]
- **Out of scope:** [State the adjacent work this demo intentionally omits.]

## Trigger and case contract

| Item | Specification |
| --- | --- |
| Trigger | [Event source and initial Flow input schema.] |
| Canonical record | [System of record, case ID, and correlation/idempotency value.] |
| Required inputs | [Documents, event data, and sensitive-data classification.] |
| Outputs | [Business result, persistence/write-back, and communication.] |

## Flow topology

Map each segment to the required reference topology. Use three or four named
sticky notes; keep the happy path left-to-right, exceptions below it, and merge
independent parallel branches before dependent work.

Include the shared Azure AI Foundry external-agent node on a labelled showcase
branch below the core path. Bind connection
`0107247a-0197-42c9-b957-05d1b722b111`, pass only static non-sensitive demo
metadata, discard the response, and rejoin without changing case state or route.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | [Title] | [Trigger, intake/API/RPA, classification/IXP, output] | [Confidence or validation path] |
| Assess and enrich | [Title] | [Agents, tools, context, API/RPA enrichment, output] | [Structured field that drives routing] |
| Decide and review | [Title] | [Decision, reviewer, safe exception route, output] | [Real expression and review outcome] |
| Act and communicate | [Title] | [Independent follow-up, write-back, communication, output] | [Parallel branches and merge] |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent | [Responsibility] | [Schema and branch-driving field] | [Tool(s), context source, guardrails] | [Resource state and fallback] |
| Coded agent | [Single visible value-add] | [Schema] | [Framework, MCP/tool purpose, evaluation] | [Resource state and fallback] |
| External agent showcase | Display Azure AI Foundry connectivity without performing domain work. | Connection-selected `agent_id`, static non-sensitive `message`, no `thread_id`; response discarded. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` and connection `0107247a-0197-42c9-b957-05d1b722b111`; no case or sensitive data. | Connection verified enabled in Playground `demos` on August 12, 2026; demo-flag branch and timeout/error rejoin the unchanged core route. |

## Data and resources

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP, if used | [Document, fields, confidence/review condition, output mapping] | [Project/model and deployed folder] | [Data classification and low-confidence route] |
| API workflow | [Business responsibility, input/output, Flow invocation] | [Connector or HTTP prerequisite] | [Failure contract] |
| RPA | [UI-only business step and input/output] | [Application/process prerequisite] | [Exception contract] |
| Persistent data | [Read/write points and canonical record] | [System owner] | [Retention/access controls] |

## Human decisions

- **Reviewer and decision:** [Role, decision authority, and information shown.]
- **Task experience:** [Coded action app or quick form; read-only and editable
  fields; required rationale.]
- **Outcomes:** [Named outcomes, timeout/escalation, returned data, and exact
  downstream Flow routes.]

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Routing safety | [Real business expression and safe exception path] | [Node/field/path] |
| Access and data | [Least privilege, connection ownership, sensitive-data handling] | [Resource or policy] |
| Agent boundaries | [Allowed tools, output guardrails, human escalation] | [Prompt/tool/task contract] |
| Resilience | [Timeouts, retries, fallback, and recovery] | [Route and owner] |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | [Case state, agent rationale, reviewer decision] | [Queryable record/trace] |
| Flow or node evaluator | [Principal business claim] | [Threshold] |
| Tool-use/trajectory evaluator | [Required agent or MCP action] | [Threshold] |
| Synthetic evaluation set | [3-5 cases: straight-through, review, exception, dependency/edge] | [Expected route and business output] |

## Success measures

- **Business proof:** [Measurable outcome the demo makes credible.]
- **Flow proof:** [How the canvas shows orchestration, routing, and actor
  contrast.]
- **Demo proof:** [What a viewer can verify in the allotted demo time.]
- **Build proof:** [Validation, resource readiness, and deployability evidence.]

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| 3-4 segment topology and canvas rules | [Mapping] | [Evidence/gap] |
| IXP/document intelligence, when relevant | [Mapping or N/A rationale] | [Evidence/gap] |
| API workflow and RPA on the intended path | [Mapping or N/A rationale] | [Evidence/gap] |
| Inline agent with a wired tool | [Mapping] | [Evidence/gap] |
| Coded agent with visible value-add | [Mapping] | [Evidence/gap] |
| Shared external-agent showcase | [Non-material branch, static input, discarded output, and connection binding] | [Evidence/gap] |
| Real business decision and safe exception | [Mapping] | [Evidence/gap] |
| Human decision and returned outcome data | [Mapping] | [Evidence/gap] |
| Purposeful parallelism and merge | [Mapping or N/A rationale] | [Evidence/gap] |
| Evaluation set and evaluator | [Mapping] | [Evidence/gap] |
| Solution boundary and delivery contract | [Mapping] | [Evidence/gap] |

## Quality rubric

Score each dimension from 0 to 3. A spec is implementation-ready only when it
scores at least 10/12, has no zero, and every gap has an owner and resolution
path.

| Dimension | 0 — absent | 1 — asserted | 2 — designed | 3 — demo-ready evidence |
| --- | --- | --- | --- | --- |
| Enterprise credibility | No consequential use case or owner. | Plausible use case but vague actor, decision, or data. | Specific role, decision, data contract, controls, and success measure. | Adds verified resource readiness and a credible operational/audit story. |
| Flow differentiation | Linear automation with no reason to use Flow. | Lists actors without a visible orchestration role. | Uses the reference segments, real routing, human decision, and purposeful parallelism. | The canvas makes multi-actor orchestration, tool use, and recovery legible at a glance. |
| Demo clarity | No narrative or proof point. | Narrative exists but the hero moment and paths are unclear. | Concise journey, named canvas segments, exception path, and observable outputs. | A viewer can follow the trigger, consequential decision, human interaction, and outcome without node-property spelunking. |
| Build feasibility | Unknown contracts or dependencies. | Major resources named but readiness is unknown. | Inputs, outputs, dependencies, fallbacks, solution boundary, and evaluation are specified. | Resource/connection readiness, validation path, deployment constraints, and owned gaps are recorded. |

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | --- | --- | --- |
| Enterprise credibility | [ ] | [ ] | [ ] |
| Flow differentiation | [ ] | [ ] | [ ] |
| Demo clarity | [ ] | [ ] | [ ] |
| Build feasibility | [ ] | [ ] | [ ] |
| **Total** | **[ /12]** | **[Ready / not ready rationale]** | **[Next action]** |
