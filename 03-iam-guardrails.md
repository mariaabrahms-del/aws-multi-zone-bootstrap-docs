# 03. AWS Organization SCPs & IAM Policy Enforcement

This section provides the policy infrastructure definitions and CLI enforcement strings required to govern a multi-account AWS environment. These guardrails prevent root-tenant configuration errors and enforce strict boundary security across all landing zones.

## 1. Service Control Policies (SCPs) Definition

Service Control Policies are administrative boundaries used to manage permissions across an entire AWS Organization. Unlike standard IAM policies, SCPs restrict actions even for the root user of a member account.

The following JSON structure models an enterprise guardrail that explicitly blocks member accounts from modifying CloudTrail logging paths or deleting central Amazon S3 logging buckets.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EnforceCentralLoggingProtection",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:UpdateTrail",
        "cloudtrail:DeleteTrail",
        "s3:DeleteBucket"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotLike": {
          "aws:PrincipalARN": "arn:aws:iam::*:role/Central-Security-Admin-Role"
        }
      }
    }
  ]
}
```

---

## 2. Policy Enforcement & Deployment Routines

To implement governance programmatically across distributed landing zones without relying on manual console actions, execute the following AWS CLI operational sequences.

### Step 1: Create the Organization Policy Element
Upload the JSON policy payload definition into your primary AWS Organization root engine.

```bash
aws organizations create-policy \
    --content file://templates/scp-logging-guardrail.json \
    --description "Enforces immutable logging state across member tenant boundaries" \
    --name "ImmutableLoggingGuardrail" \
    --type SERVICE_CONTROL_POLICY \
    --profile landing-zone-admin
```

*Extract the returned `"PolicyId"` (e.g., `p-1a2b3c4d5e`) from the terminal response matrix to use in the attachment sequence below.*

### Step 2: Query Target Account OUs (Organizational Units)
Identify the target account or container block ID where you want to attach the compliance boundary.

```bash
aws organizations list-roots \
    --query "Roots[*].Id" \
    --output text
```

### Step 3: Attach the Policy Engine to the Target Tenant Container
Enforce the compliance boundary block across your target organizational unit to instantly activate protection layers.

```bash
aws organizations attach-policy \
    --policy-id p-1a2b3c4d5e \
    --target-id ou-a1b2-c3d4e5f6 \
    --profile landing-zone-admin
```

---

## 3. Compliance and Posture Auditing

Run this operational command sequence to dynamically audit which control policies are actively throttling or governing a member account environment.

```bash
aws organizations list-policies-for-target \
    --target-id 112233445566 \
    --filter SERVICE_CONTROL_POLICY \
    --query "Policies[*].{PolicyName:Name,PolicyId:Id,Type:Type}" \
    --output table
```

### Expected Audit Response Interface:
```text
----------------------------------------------------------------------

|                       ListPoliciesForTarget                        |
+---------------------------+----------------+-----------------------+

|         PolicyName        |    PolicyId    |         Type          |
+---------------------------+----------------+-----------------------+

|  FullAWSAccess            |  p-FullAccess  | SERVICE_CONTROL_POLICY|
|  ImmutableLoggingGuardrail|  p-1a2b3c4d5e  | SERVICE_CONTROL_POLICY|
+---------------------------+----------------+-----------------------+
```
