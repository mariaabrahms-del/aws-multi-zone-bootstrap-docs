# 02. Hub-and-Spoke CLI Initialization & Route Audit

This reference runbook tracks the initialization sequence for deploying cross-account, multi-zone networking components via the AWS CLI. Ensure you have run through the [Prerequisites](./01-environment-prerequisites.md) manual before executing these routines.

## 1. Establishing Core Core Infrastructure Targets

The primary architecture uses a central **Hub VPC** acting as the network egress machine, coupled to isolated **Spoke Tenant VPCs** utilizing AWS Transit Gateway for routing.

### Step 1: Query Available Availability Zones
Ensure your active landing zone target matches resource requirements across multiple availability zones within your cloud operations target.

```bash
aws ec2 describe-availability-zones \
    --region us-west-2 \
    --query "AvailabilityZones[?State=='available'].ZoneName" \
    --output table
```

### Step 2: Provision the Central Transit Gateway Routing Element
Deploy the primary routing transit block within your infrastructure core network. Tag this resource explicitly to register it with internal auditing frameworks.

```bash
aws ec2 create-transit-gateway \
    --description "Core-Hub-Transit-Gateway" \
    --options AmazonSideAsn=64512,AutoAcceptSharedAttachments=enable \
    --tag-specifications "ResourceType=transit-gateway,Tags=[{Key=Environment,Value=Production},{Key=Owner,Value=DevOps-Core}]" \
    --profile landing-zone-admin
```

*Save the returned `"TransitGatewayId"` (e.g., `tgw-01a2b3c4d5e6f7g8h`) to pass down into subsequent tenant route definitions.*

---

## 2. Tenant Environment Route Auditing

To eliminate overlapping CIDR blocks across tenant accounts, execute a terminal query to map out routing boundaries before creating a new spoke attachment.

```bash
aws ec2 describe-vpcs \
    --filters "Name=tag:Environment,Values=Production" \
    --query "Vpcs[*].{VpcId:VpcId,CidrBlock:CidrBlock,Name:Tags[?Key=='Name'].Value | [0]}" \
    --output json
```

---

## 3. Operational Troubleshooting Registry

When testing network connectivity across automated multi-zone pipelines, your workstation terminal might encounter these environment issues. Use the remediation matrix below:

### Symptom: `An error occurred (AccessDenied) when calling the CreateTransitGateway operation`
*   **Root Cause:** Your local `AWS_PROFILE` context has timed out or lacks the necessary administrative permissions to adjust secure infrastructure components.
*   **Remediation:** 
    1. Re-authenticate your token pool: `aws sso login --profile landing-zone-admin`
    2. Confirm IAM permissions mapping: `aws sts get-caller-identity`

### Symptom: `Transit Gateway state remains permanently stuck in [pending] posture`
*   **Root Cause:** A route mapping conflict or missing dependent IAM permission blocks are preventing the resource creation from settling inside the designated AWS Zone boundary.
*   **Remediation:** Run a detailed status scan on the asset ID to view internal status strings:
    ```bash
    aws ec2 describe-transit-gateways --transit-gateway-ids tgw-YOUR-ID-HERE
    ```
