# Risk Rules

> **Status: draft.** These are the candidate rules for the first version. Each rule is validated against a real, redacted CloudTrail event stored in `tests/fixtures/` before it is implemented, and every rule has tests. Rules can be dropped, merged, or re-rated as testing shows what the events actually contain.

## How classification works

- Rules are **deterministic**: the same event always produces the same result.
- Each event is checked against every rule. The highest matching severity wins.
- Write events that match no rule are recorded as **Low**.
- The language model is never involved in classification. It only writes the explanation for events a rule has flagged.

## Severity scale

| Severity | Meaning |
|----------|---------|
| **High** | Exposes data or resources publicly, disables security visibility, or grants broad privilege. Someone should look now. |
| **Medium** | Creates lasting access or destroys resources. Worth reviewing. |
| **Low** | An ordinary write action with no notable risk signal. |

## Rules

| ID | Rule | Trigger (CloudTrail event, condition) | Severity | Why it matters |
|----|------|---------------------------------------|----------|----------------|
| R01 | Security group opened to the internet | `ec2:AuthorizeSecurityGroupIngress` with source `0.0.0.0/0` or `::/0`; High for all ports or admin/database ports, Medium otherwise | High / Medium | Makes a resource reachable by anyone on the internet |
| R02 | S3 public access block removed | `s3:DeletePublicAccessBlock`, or `s3:PutPublicAccessBlock` with any protection turned off | High | Removes the safety net that prevents accidental public buckets |
| R03 | S3 bucket made public | `s3:PutBucketPolicy` granting a wildcard principal without restricting conditions, or `s3:PutBucketAcl` granting access to all users | High | Exposes bucket contents to the internet |
| R04 | Audit logging disabled | `cloudtrail:StopLogging`, `cloudtrail:DeleteTrail` | High | Blinds anyone trying to investigate later |
| R05 | Security service disabled | `guardduty:DeleteDetector`, `securityhub:DisableSecurityHub`, `config:StopConfigurationRecorder`, `config:DeleteConfigurationRecorder`, `ec2:DeleteFlowLogs` | High | Turns off detection and monitoring |
| R06 | Administrator privilege granted | `iam:Attach*Policy` with the AWS-managed administrator policy, or a policy written with wildcard actions on wildcard resources (`PutRolePolicy`, `PutUserPolicy`, `CreatePolicyVersion`) | High | Gives an identity power over the entire account |
| R07 | Role trust widened | `iam:UpdateAssumeRolePolicy` allowing a wildcard or an external account | High | Lets another party take on the role |
| R08 | Long-lived credential created | `iam:CreateAccessKey`, `iam:CreateLoginProfile` | Medium | Creates access that persists beyond the session that made it |
| R09 | KMS key disabled or scheduled for deletion | `kms:DisableKey`, `kms:ScheduleKeyDeletion` | High | Can make encrypted data permanently unreadable |
| R10 | Data or compute resource deleted | `s3:DeleteBucket`, `rds:DeleteDBInstance`, `rds:DeleteDBCluster`, `dynamodb:DeleteTable`, `ec2:TerminateInstances` | Medium | Destroys resources that may be hard to recover |

## Design notes

- **Start small.** Ten rules that work and are tested beat fifty that half work. If time runs short, drop from the bottom and keep R01 to R07.
- **Conditions matter.** Several rules depend on the *contents* of the request (for example, the CIDR range or the policy document), not just the event name. Those fields must be confirmed in real events first.
- **Attribution is separate from severity.** A risky action is risky whoever did it. The rules score the action; the ingest step attributes it to an actor.
- **Suggested fixes are advice, not actions.** For each finding, the app shows a recommended remediation in plain English. It never applies one.

## Testing

For each rule:

1. Save a real, redacted CloudTrail event that should trigger it in `tests/fixtures/`.
2. Save a near-miss that should not trigger it (for example, a security group opened to a private range).
3. Assert the severity for both.
4. Keep a fixture for a benign write event, which should classify as Low.