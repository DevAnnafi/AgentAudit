# Process Log

A running record of how AgentAudit was built: what I did, what the coding agent did, what broke, and what I decided. The hackathon rubric asks for the development process and how the agent was used, so this file is written as I go, not reconstructed at the end.

**Redaction convention:** nothing in this repo contains real account IDs, ARNs, IP addresses, or access keys. Use the placeholder account ID `111122223333` (the one AWS uses in its own documentation) and `REDACTED` elsewhere. Redact screenshots before committing them.

**Two different "Claudes" appear in this log.**
- **Planning chat:** conversations in the Claude chat app, used for scoping, reading docs together, and reviewing my thinking. It has no access to my AWS account.
- **Coding agent:** Claude Code, connected to my AWS account through the Agent Toolkit for AWS. This is the agent whose actions AgentAudit records.

---

## Entry template

Copy this for each working session.

```
## Entry N: <short title>
**Date / time:**
**Goal:**

**What I did:**
-

**What the coding agent did:**
-

**What broke or surprised me:**
-

**Decisions and why:**
-

**Evidence saved (paths in docs/proof/):**
-

**Next:**
-
```

---

## Entry 1: Rules and scope
**Date:** 2026-09-18
**Goal:** Understand exactly what the hackathon requires before building anything.

**What I did:**
- Read the full Terms and Conditions for the AWS Zero to Shipped Hackathon.
- Submission period: September 18, 2026 (9:00 a.m. PT) to October 2, 2026 (11:59 p.m. PT). Late submissions are grounds for disqualification.
- Required for every entry: a live, publicly reachable app on AWS; documented proof of a coding agent connected to the AWS console; a published Builder Center project describing the app, the development process, and how the agent was used; one of five categories and one track (Community or Startup).
- Ship gate is pass/fail: the app must be live and reachable at evaluation time, by both the AI scoring system and human judges. Judging runs from the week of October 5 through the week of October 19, so the app has to stay up and affordable through then.
- Scoring: four equal criteria at 25% each: Technical Innovation & Originality, Implementation Quality, Community/Market Impact, Creativity & Storytelling. Top 100 by composite score advance to human judging; the panel picks 5 winners.
- Limit of one entry per person.
- The rules do not specify which coding agent to use or define what counts as "documented proof."

**Decisions and why:**
- Project concept: a flight recorder for coding agents on AWS (working name was "AgentAudit"; renamed, see Entry 3). It records what a coding agent actually did in an account, scores each action's risk with deterministic rules, and explains flagged actions in plain English.
- Category and track: `#commercial-potential`, `#startup`.
- Keep evidence of the agent connection from day one, since the proof format is undefined.

**Next:**
- Check the Builder Center submission form for a proof-of-connection field and its format.

---

## Entry 2: How the agent connects to AWS
**Date:** 2026-09-18
**Goal:** Work out the correct way to connect Claude Code to AWS.

**What I did:**
- Read the AWS Security Blog post on OAuth support for the AWS MCP Server (July 2026). Key points: an authorized agent acts with the permissions of the identity that signed in, and OAuth sessions are recorded in CloudTrail with a sign-in session ARN.
- Read the Builder Center article "Connect your AI coding agent to AWS" (September 17, 2026). It describes the Agent Toolkit for AWS: the AWS MCP Server plus AWS skills. Setup path: AWS CLI 2.35.0 or later, `aws login`, then the `aws configure agent-toolkit` wizard, then restart the agent and verify by asking "What AWS Regions are available?"
- The article states the MCP Server uses the permissions of the signed-in identity for both read and write calls, and that AWS supports permissions specific to agent requests.

**What broke or surprised me:**
- The two sources describe different sign-in mechanisms (OAuth browser flow vs. `aws login` plus wizard). I do not yet know which authentication the wizard configures, so I cannot assume the sign-in session ARN appears on ordinary API calls.

**Decisions and why:**
- Use the manual setup path, not the "Get setup prompt" button, so I can see exactly what changes.
- Give the agent its own dedicated, narrowly scoped AWS identity. If it shared my credentials, its actions would be indistinguishable from mine in CloudTrail, and attribution is the core of the product.

**Open questions (answer with a real CloudTrail event):**
- Which field identifies who made the call?
- Can I tell the agent's identity apart from mine?
- Is there any marker that a call came through the MCP Server?
- Which region do the events appear in, and where must the EventBridge rule live?

**Next:**
- Read the IAM-for-managed-MCP-servers doc and the CloudTrail logging page for the MCP Server.

---

## Entry 3: Name check and rename
**Date:** 2026-09-18
**Goal:** Make sure the project name is not already in use before creating the repo.

**What I did:**
- Searched for the working name "AgentAudit" and for "AgentTrail" (a second candidate).
- "AgentAudit" is already used by several unrelated projects, including an npm CLI and GitHub Action for scanning MCP servers, an AI agent regression-testing platform, a research paper published in September 2026, and a sample in AWS's samples collection whose deployment uses "agentaudit" as its stack name. "AgentTrail" is also used by several existing projects.
- I only skimmed the AWS sample from search results. TODO: read it properly and write down how this project differs (it appears to be about cost analysis, but confirm).

**Decisions and why:**
- Final name: `{{NAME}}`.
- Why: TODO. If this is a new name, note that it avoids confusion with the existing tools above and lets judges and the AI scorer find one clear project when they search. If I kept "AgentAudit", note why, and how this project is distinct from the others.
- Names checked before committing: TODO (GitHub search, PyPI, npm, domain).

**Next:**
- Create the repo with the final name.

---

## Entry 4: (fill in) Connection and first CloudTrail event
**Date:**
**Goal:** Connect Claude Code under its own identity and inspect a real event.

Suggested contents:
- Which identity type and setup path I used, and why
- Screenshots saved: console banner, wizard output, "What AWS Regions are available?" test, installed skills
- The first action the agent took (a tagged S3 bucket), and the CloudTrail event it produced
- Answers to the four open questions above

---

## Architecture decisions

Record each significant decision here as it is made, with the alternative I rejected.

| Date | Decision | Alternatives considered | Why |
|------|----------|-------------------------|-----|
| 2026-09-18 | Risk scoring is deterministic rules; the LLM only writes explanations | LLM-scored risk | Decisions must be explainable and testable; the model should not decide what is risky |
| 2026-09-18 | Detection and explanation only, no auto-remediation | Automatic containment | Scope for a two-week build; suggested fixes are shown, not applied |
| 2026-09-18 | Read-only public app with a demo mode | Login-gated app | Judges and the AI scorer must see meaningful data with no credentials |

---

## Where the agent got things wrong

Keep an honest list. It strengthens the "how the agent was used" story.

- (none yet)

---

## Submission checklist

- [ ] Builder Center profile exists (builder.aws.com), age 18+
- [ ] Repo public, README complete, license present
- [ ] App live at a public URL with demo mode
- [ ] Proof of agent connection saved and referenced in the Builder Center project
- [ ] Builder Center project published: description, development process, how the agent was used, live URL
- [ ] Tags added: one category tag and one lane tag
- [ ] Submitted at least a day early (deadline: October 2, 2026, 11:59 p.m. PT)
- [ ] Billing alert active; app affordable through late October
- [ ] No real account IDs, ARNs, IPs, or keys anywhere in the repo or screenshots