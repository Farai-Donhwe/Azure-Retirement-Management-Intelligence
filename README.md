# Azure Retirement Management Intelligence

Programmatically detect upcoming Azure service/feature retirements across subscriptions, assess impact to resources, generate change recommendations and best practices, and automate incident/change workflows in ServiceNow.

## Architecture

```mermaid
---
title: Azure Retirement Intelligence + ServiceNow Change Workflow
---
flowchart LR
    %% ═══ Azure Signal Sources ═══
    subgraph azure ["Microsoft Azure"]
        advisor_wb["Advisor: Service<br/>Retirement Workbook"]
        advisor_api[/"Advisor APIs"/]
        arg[/"Azure Resource Graph"/]
        svc_health["Service Health<br/>Retirement Announcements"]
        itsm["Azure Monitor:<br/>ITSM Connector"]
        advisor_wb -.- advisor_api
        advisor_wb -.- arg
    end

    %% ═══ Automation & Orchestration ═══
    subgraph auto ["Automation & Agent Orchestration"]
        sched(["Scheduler - Daily/Hourly"])
        collector["Retirement Collector"]
        analyzer["Impact Analyzer"]
        bpe["Best Practice &<br/>Change Guidance Engine"]
        agent{{"Automation Agent<br/>Ticketing + Change"}}
        store[("Retirement<br/>Intelligence Store")]
        dash["Reporting Dashboard<br/>Ops / Exec / FinOps"]
    end

    %% ═══ ServiceNow ═══
    subgraph snow ["ServiceNow"]
        incident["Incident"]
        chg["Change - ITIL/CAB"]
        cmdb["CMDB<br/>Apps / Services / Owners"]
    end

    %% ═══ Identity & Secrets ═══
    subgraph idp ["Identity & Secrets"]
        entra["Microsoft Entra ID<br/>App / Managed Identity"]
        vault[("Secrets Store<br/>Tokens / Keys")]
    end

    %% ═══ Personas ═══
    ops(["Cloud Ops / SRE"])
    cam(["Change Manager / CAB"])
    apo(["Application Owner"])
    fin(["FinOps / Cost Governance"])

    %% ═══ Core Data Flows ═══
    sched -->|"Trigger collection"| collector
    collector -->|"Pull retirement recs"| advisor_api
    collector -->|"Query retirements"| arg
    collector -->|"Ingest announcements"| svc_health
    collector -->|"Persist data"| store
    analyzer <-->|"Read/write impact"| store
    analyzer -->|"Resolve owners"| cmdb
    analyzer --> bpe
    bpe -->|"Change checklist + plan"| agent
    agent -->|"Create/Update Incident"| incident
    agent -->|"Create/Update Change"| chg
    itsm -.->|"Alerts to Incident"| incident
    store -->|"BI / reporting"| dash

    %% ═══ Persona Touchpoints ═══
    dash -.-> ops
    dash -.-> fin
    chg -.-> cam
    incident -.-> apo

    %% ═══ Identity Flows ═══
    entra -. Auth .-> collector
    vault -. Credentials .-> agent

    %% ═══ Styles ═══
    classDef azureNode fill:#E6F2FF,stroke:#0078D4,stroke-width:2px,color:#003A6C
    classDef autoNode fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B5E20
    classDef snowNode fill:#FFF3E0,stroke:#E65100,stroke-width:2px,color:#BF360C
    classDef idpNode fill:#FFF8E1,stroke:#F57F17,stroke-width:2px,color:#E65100
    classDef persona fill:#F3E5F5,stroke:#7B1FA2,stroke-width:1px,color:#4A148C

    class advisor_wb,advisor_api,arg,svc_health,itsm azureNode
    class sched,collector,analyzer,bpe,agent,store,dash autoNode
    class incident,chg,cmdb snowNode
    class entra,vault idpNode
    class ops,cam,apo,fin persona
```

## Components

### Azure Signal Sources

| Component | Description |
|-----------|-------------|
| **Advisor: Service Retirement Workbook** | Centralized view of service retirements and impacted resources |
| **Azure Advisor APIs** | Automation pull for retirement recommendations |
| **Azure Resource Graph** | Alternative query path for retirements data |
| **Azure Service Health** | Portal notifications and retirement announcements |
| **ITSM Connector** | Azure Monitor integration with ServiceNow (note: ITSM actions path deprecated since Sept 2022) |

### Automation & Agent Orchestration

| Component | Responsibilities |
|-----------|-----------------|
| **Scheduler** | Triggers discovery and correlation runs (daily/hourly) |
| **Retirement Collector** | Query retirements across subscriptions, normalize/deduplicate events, store results |
| **Impact Analyzer** | Map retirements to resources/apps/owners, determine urgency by retirement date + blast radius, enrich with migration guidance |
| **Best Practice & Change Guidance Engine** | Generate change plan checklists (risk, testing, rollback, comms), suggest governance actions, produce remediation runbook outlines |
| **Automation Agent** | Decide Incident vs Change vs Problem, create/update ServiceNow tickets, attach impact analysis and recommendations, track status and re-notify on approaching deadlines |
| **Retirement Intelligence Store** | Persistence for retirements, impacted resources, ownership mappings, ticket links, and change plans |
| **Reporting Dashboard** | Top upcoming retirements, subscriptions at risk, trend + completion rate, cost/risk posture views |

### ServiceNow

| Module | Purpose |
|--------|---------|
| **Incident** | Break/fix urgency tickets |
| **Change (ITIL/CAB)** | Standard change requests with CAB workflow |
| **CMDB** | Application, service, and owner resolution |

### Identity & Secrets

| Component | Responsibilities |
|-----------|-----------------|
| **Microsoft Entra ID** | Auth for Azure APIs, least privilege across subscriptions |
| **Secrets Store** | Store ServiceNow credentials/tokens, rotate secrets |

## Personas

| Persona | Role |
|---------|------|
| **Cloud Ops / SRE** | Consumes dashboards and acts on retirement remediation |
| **Change Manager / CAB** | Reviews and approves change requests |
| **Application Owner** | Receives incident notifications for affected applications |
| **FinOps / Cost Governance** | Monitors cost and risk posture from retirements |

## Security Principles

- Least privilege across subscriptions via Entra ID app or managed identity
- No long-lived secrets in code; store tokens in secrets vault
- Audit trail: ticket IDs + change approvals linked back to retirement IDs

### Security Controls

| Control | Description |
|---------|-------------|
| **RBAC** | Scoped at Management Group / Subscription level |
| **Secret Rotation** | OAuth token management for ServiceNow |
| **Observability** | Logging + alerting for failed ticket creation / workflow execution |

## Outputs

- Ranked list of upcoming retirements by date + impacted resources
- ServiceNow Incident/Change records with impact + recommended actions
- Executive dashboard view of risk posture and remediation progress
