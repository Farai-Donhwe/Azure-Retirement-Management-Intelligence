# Security Design

This document outlines the security principles, controls, and requirements for the Azure Retirement Management Intelligence system.

## Principles

1. **Least Privilege** — All Azure API access is scoped via Entra ID app registrations or managed identities with the minimum required permissions at the Management Group or Subscription level.
2. **No Long-Lived Secrets in Code** — All credentials (ServiceNow tokens, API keys) are stored in a secrets vault and never embedded in source code or configuration files.
3. **Audit Trail** — Every ServiceNow ticket ID and change approval is linked back to its originating retirement ID, providing full traceability.

## Security Controls

### S1: RBAC Scoping

- Azure RBAC roles are assigned at the Management Group or Subscription scope.
- The Retirement Collector service principal requires `Reader` access across target subscriptions.
- The Impact Analyzer may need `Reader` access to resource metadata.
- No write permissions to Azure resources are required.

### S2: Secret Rotation & OAuth Token Management

- ServiceNow credentials are stored in Azure Key Vault (or equivalent secrets store).
- OAuth tokens are refreshed automatically; long-lived tokens are not used.
- Secret rotation policies are enforced with alerting on expiration.

### S3: Logging & Alerting

- All ticket creation and workflow execution attempts are logged.
- Failed operations trigger alerts to the Cloud Ops / SRE team.
- Logs are retained in compliance with organizational retention policies.

## Identity Architecture

| Component | Identity Type | Purpose |
|-----------|--------------|---------|
| Retirement Collector | Managed Identity or App Registration | Authenticate to Azure Advisor APIs, Resource Graph, Service Health |
| Automation Agent | Service Principal | Authenticate to ServiceNow REST APIs using OAuth tokens from vault |
| Reporting Dashboard | User Identity (Entra ID) | Role-based access for Ops, FinOps, and Executive personas |

## Threat Model Considerations

| Threat | Mitigation |
|--------|------------|
| Unauthorized access to retirement data | RBAC + Entra ID authentication |
| ServiceNow credential exposure | Secrets stored in vault with rotation |
| Stale or missed retirements | Scheduler cadence + alerting on collection failures |
| Tampering with ticket data | Audit trail linking retirements to tickets |
| Lateral movement from compromised collector | Least privilege RBAC, no write access to resources |
