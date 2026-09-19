# AgentAudit

**A flight recorder for coding agents on AWS.**

- **Live demo:** `TODO: public URL`
- **Category:** `#commercial-potential`
- **Track:** `#startup`

Built for the AWS Zero to Shipped Hackathon.

> Status: work in progress. Sections marked TODO are filled in as the build progresses.

---

## The problem

Coding agents can now connect directly to an AWS account and make real changes with the permissions of whatever identity they use. The usual safeguard is preventive: scope the agent's IAM permissions so it can only do so much.

Prevention answers "what could the agent do?" It does not answer:

- What did the agent actually do?
- Which of those changes were risky?
- Which agent session did each change come from?
- Why does a given change matter, in words a non-expert can act on?

AgentAudit answers those questions.

## What it does

- **Captures** write actions in an AWS account from CloudTrail via EventBridge.
- **Attributes** each action to the identity or session that made it, so agent activity can be separated from human activity.
- **Scores risk** with a small set of deterministic, tested rules (public exposure, privilege escalation, disabled logging, deleted resources).
- **Explains** each flagged action in plain English, with a suggested fix. The language model writes the explanation only; it never decides the risk level.
- **Shows** a live feed, a per-session timeline, and a risk summary.
- **Demo mode** with pre-seeded events, so the public site is meaningful without any credentials.

| Feature | Status |
|---------|--------|
| Event capture | TODO |
| Attribution | TODO |
| Risk rules | TODO |
| Explanations | TODO |
| Frontend | TODO |
| Demo mode | TODO |

## Architecture

`TODO: diagram (docs/architecture.md)`

Planned components:

- **Event capture:** CloudTrail management events routed through EventBridge, filtered to write actions
- **Risk engine:** AWS Lambda applying the rules in [`docs/risk-rules.md`](docs/risk-rules.md)
- **Explanation layer:** Amazon Bedrock, given a flagged event and asked for a short plain-English explanation and suggested fix
- **Storage:** Amazon DynamoDB (events, findings, session timelines) with a TTL so the app stays cheap while it runs
- **API and frontend:** Amazon API Gateway, with a static site served from S3 and CloudFront

## Risk rules

Each rule is documented, tested, and explained in [`docs/risk-rules.md`](docs/risk-rules.md).

| ID | Rule | Category | Severity | Why it matters |
|----|------|----------|----------|----------------|
| TODO | TODO | Public exposure | TODO | TODO |
| TODO | TODO | Privilege escalation | TODO | TODO |
| TODO | TODO | Disabled logging | TODO | TODO |
| TODO | TODO | Deleted resources | TODO | TODO |

## How the coding agent was used

*The hackathon requires this section. Fill it in from `docs/process-log.md`.*

- **Agent:** Claude Code, connected to AWS through the Agent Toolkit for AWS (AWS MCP Server plus AWS skills).
- **Identity:** the agent uses its own dedicated, narrowly scoped AWS identity, separate from my personal credentials, so its actions can be attributed. `TODO: describe the identity type`
- **Connection proof:** see [`docs/proof/`](docs/proof/) (screenshots and CloudTrail excerpts, with account details redacted).
- **What the agent did:** `TODO`
- **What I wrote myself:** `TODO`
- **Where the agent got things wrong, and how I caught it:** `TODO`
- **A project that audits its own builder:** {{NAME}} records the actions the agent took while building it, and that activity is part of the demo data. `TODO: confirm once true`

The full timeline is in [`docs/process-log.md`](docs/process-log.md).

## Security and privacy design

- **Read-only public app.** The public site can display data but cannot change anything in any account.
- **Redaction.** Account IDs, ARNs, and IP addresses are redacted in everything the public site shows. `TODO: describe how`
- **No credentials in the repo.** See `.env.example` for the configuration a deployment needs.
- **Least privilege.** Every Lambda role is scoped to only what it uses. `TODO: confirm`
- **Bounded storage.** Stored events expire after a set time.

## Getting started

Prerequisites: `TODO` (AWS account, AWS CLI, your IaC tool)

```
TODO: deploy instructions
```

## Tests

`TODO: how to run the tests, and what they cover`

## Limitations and roadmap

Known limitations, stated honestly:

- Detection and explanation only. AgentAudit does not block or undo any action.
- `TODO`

Where this could go as a product:

- `TODO: multi-account support`
- `TODO: alerting integrations`
- `TODO: suggested containment policies for a single agent session`

## License

MIT. See [`LICENSE`](LICENSE).