A10: Secure Infrastructure Proposal

Scenario: A mid-sized SaaS company expanding into the education market on Microsoft Azure.
The company currently has a basic cloud setup with minimal security. It needs to support enterprise SSO, least-privilege access, audit log retention, compliance readiness, and a weekly deployment cadence.

Part A — Architecture Diagram
I chose to stay on Microsoft Azure so that identity, logging, policy, and detection are all in one stack and easier to explain to an auditor.
The diagram below shows the full security flow from login to incident response.

![alt text](image.png)

How to read the flow


| Step | Meaning |
| --- | --- |
| Users log in | Schools and organizations use their existing accounts via SSO. All logins go through Entra ID. |
| Identity check | Entra ID confirms who the user is and enforces MFA. Conditional Access policies block risky sign-ins. |
| Access control | RBAC roles decide what each user can see or do. Admins use PIM for temporary elevated access only. |
| App + resources | Services authenticate using managed identities so there are no hardcoded passwords in code. |
| All activity is logged | Every action from every service lands in Log Analytics — one central place for audit and search. |
| Policy + IaC | Azure Policy and Terraform templates prevent misconfigs before and after deployment. |
| Alerts fire | Sentinel monitors the logs and fires alerts when something looks suspicious. |
| Contain + fix | Sessions are revoked, IPs are blocked, runbooks are updated to prevent it from happening again. |

## Part B — Compliance Mapping Table

| Requirement | What we do | Tool | Proof for auditor |
| --- | --- | --- | --- |
| Only authorized users can log in | SSO + MFA enforced for every login | Azure Entra ID | Sign-in logs, MFA registration report |
| Admins are tracked and accountable | Named admin accounts only, temporary access that expires | Entra PIM | Elevation logs with user + timestamp |
| All activity is recorded | Every log from every service goes to one central place | Log Analytics | Retention settings, sample log export |
| Suspicious activity is caught | Alerts trigger when something looks wrong (unusual location, failed logins, etc.) | Microsoft Sentinel | Alert rule list, incident history |
| Bad configs are blocked before deploy | Policy rules and IaC templates enforced in the pipeline | Azure Policy + Terraform | Policy compliance report, failed pipeline example |

## Part C — Incident Response Outline

**Scenario:** A user account successfully logs in from a country the account has never accessed from before, outside of business hours.

| Phase | What happens |
| --- | --- |
| **Detection** | Sentinel fires an alert from a KQL rule that flags a successful login from an unusual location combined with an atypical sign-in time. The on-call person gets notified through the normal alerting path. |
| **Evidence** | Pull the sign-in logs from Entra ID (IP address, location, device, MFA status), the audit logs showing what the user accessed after logging in, and the Sentinel incident record. Save copies to immutable storage in case it needs legal or customer review. |
| **Containment** | Immediately revoke all active sessions for that account. Disable the account in Entra ID temporarily. Block the suspicious IP at the network edge. Notify the account owner to verify if it was them. |
| **Remediation** | Add a Conditional Access named location policy to block logins from non-approved countries. Require the user to re-verify their identity and re-enroll MFA before re-enabling the account. Update the KQL detection rule if it was too slow to catch it. Document the incident in the runbook. |

