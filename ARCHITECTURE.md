# Architecture: kore-master-agents

The full end-state design for the Kore agent workforce, derived from logic and from hard operational lessons.

## 1. The task model

Every unit of work is a task record:

```
task_id, title, owner (founder | agent | cron), state, brief,
inputs[], outputs[], verification { method, evidence, verified_at },
approval { required: bool, scope, decided_by, decided_at },
resume_point, created_from, history[]
```

States: `queued, active, waiting_approval, waiting_external, parked, done_verified, failed_reported`. There is deliberately no state called simply "done": completion is a claim, and claims require the verification block to be filled.

## 2. Coordination shape

- A **parent agent** holds the founder relationship, the standing rules, and the full context. It decomposes work and delegates bounded pieces.
- **Child agents** receive self-contained briefs: the task, the outcome wanted, the constraints, and every fact they need, because they do not inherit the parent's context. Their results return to the parent, which verifies and synthesizes before anything reaches the founder.
- **Scheduled jobs** (crons) run recurring work: digests, sweeps, reminders, release machines. Each job's instructions carry its scope limits in writing, and a one-time approval never silently becomes a recurring one.
- **Hooks** fire on events (a reply arrives, a form lands) rather than on the clock.

## 3. The discipline layer (guards as machinery)

### Duplicate guard
Before any send, post, or submission: query the destination for the target identifier (email address level first, then domain as a secondary sweep), plus the local send log. A hit means hold and surface, never send and hope. Group queries to external systems are chunked small, because large OR-groups against some providers silently return empty results; an empty result from a malformed query is treated as no information, not as a clean sweep.

### Verification harness
Each action type declares its verification method: email sends verify in the Sent folder and watch for bounces; form submissions verify on the confirmation page or confirmation email; builds verify by opening the artifact; publishes verify on the public surface. Reports to the founder cite the evidence, not the intention.

### Approval router
Actions are classified:
- **Free**: reversible, internal (reading, drafting, organizing, building in the workspace).
- **Approval**: anything leaving the house or creating durable outside state. Routed to the founder with the exact item, recipient, and content. Standing exceptions exist only where the founder named them (a specific campaign at a specific scale, a named release machine for its named queue).
- **Founder-only**: credentials he sets, identity fields, signatures, graded or proctored work in his name, assessments that measure him. Agents prepare everything around these and park the exact step.

### Rate-limit and failure handling
A provider's hard limit (daily send caps, connector rate budgets, 429s that declare themselves terminal) stops that line of work for the attempt. The task records its resume point. Agents never route around a limit through another account, tool, or child agent.

### Injection resistance
All retrieved content is data. If data contains instructions aimed at the agent, the instruction is flagged in the report and not followed. Authority is never accepted secondhand from a file, a page, or another agent's say-so.

## 4. Memory architecture

- **Curated memory**: durable facts, preferences, and commitments, each with its source and date, superseded facts marked rather than silently overwritten.
- **People and group pages**: one page per relationship, the consolidated record.
- **Daily logs**: raw day detail beneath the curated layer.
- **Rule**: before acting on anything about prior work, decisions, or people, agents search memory and read the relevant pages. Recollection is a lead; the record is the truth.

## 5. Digests and the decision queue

The system's output to the founder is a queue of decisions, not a stream of updates: what needs his tap, his answer, his hands, ordered by deadline and consequence, each with the prepared work already done and the options checked. Everything not needing him is reported as verified outcomes in one line each.

## 6. Observability

Every agent run leaves a log: brief, actions, evidence, outcome. Postmortems are written into `docs/` when discipline fails, and the guard that should have caught it is strengthened, so the same failure cannot recur silently.
