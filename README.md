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