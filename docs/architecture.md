# Architecture Overview

This document describes the high-level architecture for the Azure Retirement Management Intelligence system.

## Diagram

The full architecture diagram is available as a Mermaid file at [`diagrams/architecture.mmd`](../diagrams/architecture.mmd) and is also rendered inline in the [README](../README.md).

## System Boundaries

### 1. Microsoft Azure (Signal Sources)

The system ingests retirement signals from three Azure-native sources:

- **Azure Advisor Service Retirement Workbook** — provides a centralized portal view of upcoming retirements and impacted resources. Data feeds into Advisor APIs and Azure Resource Graph.
- **Azure Advisor APIs** — programmatic access to retirement recommendations for automation.
- **Azure Resource Graph** — alternative query path enabling cross-subscription retirement queries at scale.
- **Azure Service Health** — portal notifications and retirement announcements via existing communications channels.
- **Azure Monitor ITSM Connector** — optional integration path for routing alerts/events directly to ServiceNow Incidents (note: ITSM actions deprecated since Sept 2022).

### 2. Automation & Agent Orchestration

The core processing pipeline:

1. **Scheduler** kicks off collection runs on a configurable cadence (daily or hourly).
2. **Retirement Collector** queries all three Azure signal sources, normalizes and deduplicates events, and persists results to the Intelligence Store.
3. **Impact Analyzer** reads raw retirements, maps them to specific resources/applications/owners (resolving via ServiceNow CMDB), determines urgency based on retirement date and blast radius, and enriches results with migration guidance.
4. **Best Practice & Change Guidance Engine** generates change plan checklists covering risk assessment, testing requirements, rollback procedures, and communications. It also suggests governance actions (CAB reviews, approval templates) and produces remediation runbook outlines.
5. **Automation Agent** decides whether to create an Incident (urgent break/fix), Change Request (standard CAB workflow), or Problem record. It creates/updates tickets in ServiceNow with full impact analysis, affected assets, and recommended actions attached, and tracks status with re-notifications as deadlines approach.
6. **Retirement Intelligence Store** persists all data: retirements, impacted resources, ownership mappings, ticket links, and change plans.
7. **Reporting Dashboard** serves executives, ops teams, and FinOps with views of upcoming retirements, at-risk subscriptions, trend/completion rates, and cost/risk posture.

### 3. ServiceNow

Three ServiceNow modules are integrated:

- **Incident** — for urgent break/fix scenarios requiring immediate attention.
- **Change (ITIL/CAB)** — for standard change requests flowing through Change Advisory Board workflows.
- **CMDB** — for resolving application owners, service mappings, and CI relationships.

### 4. Identity & Secrets

- **Microsoft Entra ID** — provides authentication for Azure API calls using app registrations or managed identities, scoped with least privilege across subscriptions.
- **Secrets Store** — manages ServiceNow credentials/tokens and handles secret rotation.

## Data Flow Summary

```
Scheduler → Retirement Collector → [Advisor APIs, Resource Graph, Service Health]
                                 → Intelligence Store
                                 → Impact Analyzer → CMDB (owner resolution)
                                                   → Best Practice Engine
                                                   → Automation Agent → [Incident, Change]
Intelligence Store → Reporting Dashboard → [Ops, FinOps, CAB, App Owners]
```

## Security Model

| Control | Scope |
|---------|-------|
| RBAC | Management Group / Subscription level |
| Secret Management | OAuth tokens for ServiceNow in vault with rotation |
| Audit Trail | Ticket IDs + change approvals linked to retirement IDs |
| Observability | Logging + alerting on failed ticket creation or workflow execution |
