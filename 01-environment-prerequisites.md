# 01. Local Workstation Environment & Tooling Prerequisites

This guide outlines the baseline workstation configurations required to safely execute multi-zone infrastructure deployments. Follow these verification steps to eliminate local environment drift before executing terminal routines.

## 1. System Dependencies Verification

Your local environment must match the following architectural runtime versions. Run the terminal strings below to verify compliance.

### AWS CLI Version 2
The deployment routines leverage multi-account named profiles and AWS IAM Identity Center configurations native to AWS CLI v2.

*   **Verification Command:**
    ```bash
    aws --version
    ```
*   **Expected Output Matrix:**
    ```text
    aws-cli/2.15.x Python/3.11.x Linux/x86_64 or Darwin/arm64
    ```
*   **Remediation:** If your local machine returns a `1.x` version or `command not found`, update your system binaries via the official [AWS Installation Portals](https://amazon.com).

### Session Manager Plugin
Required for secure, borderless terminal sessions into private subnet nodes without managing explicit SSH keys.

*   **Verification Command:**
    ```bash
    session-manager-plugin --version
    ```
*   **Expected Output:** `1.2.x` or higher.

---

## 2. Secure Profile and Credential Scopes

Never use hardcoded root account credentials or export plaintext AWS Access Keys into your shell history. This architecture forces tokenization using short-lived AWS IAM Identity Center (formerly AWS SSO) sessions.

### Initializing the Session
Execute the authentication sequence to point to your designated multi-account start portal:

```bash
aws sso login --profile landing-zone-admin
```

### Exporting Session Tokens Dynamically
For third-party developer scripts or temporary environment utilities that do not natively parse named profile config files, parse and pass the short-lived access block safely to your execution variables:

```bash
export AWS_PROFILE="landing-zone-admin"
export AWS_REGION="us-west-2"
```

Verify your active identity posture before touching network states:
```bash
aws sts get-caller-identity
```

---

## 3. Pre-Flight Workstation Checklist

| Target Dependency | Required Condition | Purpose |
| :--- | :--- | :--- |
| `jq` | Installed (`jq --version`) | Used to clean and extract structured JSON elements out of massive cloud outputs. |
| `git` | Installed (`git version`) | Manages documentation reviews and tracking code commits inside Docs-as-Code workflows. |
| Outbound Port 443 | Open | Essential for connecting securely to AWS endpoint endpoints over HTTPS. |
