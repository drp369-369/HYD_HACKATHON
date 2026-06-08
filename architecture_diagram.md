# MuleShield AI: Enterprise-Grade Architecture
This document details the architectural layout of the **MuleShield AI** platform, featuring visual Mermaid diagrams to demonstrate data flows, governance checks, investigator workflows, and network intelligence databases.

---

## SECTION 1: 10-Layer Enterprise Architecture

MuleShield AI is built as a microservices-based system divided into 10 distinct layers. Crucially, the **Governance** and **Compliance** layers sit above the Core AI modules to monitor model execution, ensure bias-free predictions, and provide standard audit trails.

```mermaid
graph TD
    %% Define styles
    classDef layerStyle fill:#f9f9f9,stroke:#333,stroke-width:2px;
    classDef govStyle fill:#ffe6cc,stroke:#d79b00,stroke-width:2px;
    classDef coreStyle fill:#dae8fc,stroke:#6c8ebf,stroke-width:2px;

    subgraph Platform_Supervision [Layer 9 & 10: Model Supervision & Trust]
        direction LR
        L10[Layer 10: Compliance Layer<br/>RBI Guidelines / AML & CFT Regulations / FATF Lists]
        L9[Layer 9: Governance Layer<br/>Model Registry / Version Lineage / Drift Monitor / Bias Checker]
        L10 <--> L9
    end

    subgraph Data_Pipeline [Layer 1 & 2: Ingestion & Feature Engineering]
        L1[Layer 1: Data Layer<br/>Core Ledgers / KYC DB / UPI Streams] --> L2[Layer 2: Feature Layer<br/>Feast Store / Velocity Features / Behavioral Personas]
    end

    subgraph Core_AI_Engine [Layer 3, 4 & 5: Prediction & Risk]
        L3[Layer 3: ML Layer<br/>Supervised XGBoost & Unsupervised Isolation Forest] --> L4[Layer 4: Risk Layer<br/>MRS Score Aggregator]
        L4 --> L5[Layer 5: Explainability Layer<br/>SHAP Waterfall Engine / NLG Explainer]
    end

    subgraph Graph_GenAI [Layer 6 & 7: Intelligence & Copilot]
        L6[Layer 6: Network Intelligence Layer<br/>Neo4j Graph Database / Community Detection]
        L7[Layer 7: AI Fraud Copilot Layer<br/>LLM Report Orchestrator / SAR Drafter]
    end

    subgraph UI_Layer [Layer 8: Front-End Case Management]
        L8[Layer 8: Investigator Layer<br/>Analyst Dashboard / Alert Action Hub]
    end

    %% Data Flow Connections
    L2 --> L3
    L2 --> L6
    
    %% Graph and ML Interaction
    L6 --> L4
    L3 --> L4
    
    %% Supervision Interception
    Platform_Supervision -.-> L3
    Platform_Supervision -.-> L4
    Platform_Supervision -.-> L5
    
    %% Decision Flow
    L5 --> L7
    L7 --> L8
    L8 --> Platform_Supervision

    %% Apply Classes
    class L9,L10 govStyle;
    class L3,L4,L5,L6,L7 coreStyle;
    class L1,L2,L8 layerStyle;
```

---

## SECTION 2: Real-Time Data Flow Architecture

The data pipeline handles both low-latency transaction validation (e.g., UPI transfers) and batch-based historical analysis.

```mermaid
sequenceDiagram
    autonumber
    participant CB as Core Banking / UPI Gateway
    participant Kafka as Kafka Event Stream
    participant Feast as Feast Real-Time Feature Store
    participant Engine as MuleShield AI Engine (XGBoost + IF)
    participant Graph as Neo4j Graph DB
    participant Cache as Redis Cache (MRS Score)
    participant Gov as Compliance & Governance Service

    CB->>Kafka: Publish Transaction Event
    Kafka->>Feast: Retrieve Historical Velocity Features
    Kafka->>Graph: Upsert Transaction Vertex & Edges
    Graph-->>Engine: Fetch PageRank / Centrality Metrics
    Feast-->>Engine: Feed Behavioral Feature Vector
    
    Note over Engine: Engine calculates MRS Risk Score & SHAP vectors
    Engine->>Gov: Validate Model Output against Governance Policies
    Gov-->>Engine: Policy Approved / Audit Trail Logged
    
    Engine->>Cache: Write Risk Score (MRS 0-100)
    Engine->>CB: Return Decision (Allow / Restrict / Freeze)
```

---

## SECTION 3: Case Management & Investigator Workflow

This diagram maps how a transaction flag is handled by the operational units of the bank, demonstrating the workflow from initial machine alert to final Suspicious Transaction Report (STR) filing.

```mermaid
graph TD
    Alert[MRS Risk Alert Triggered] --> ScoreCheck{Risk Score MRS}
    
    ScoreCheck -->|MRS 0-30: Safe| Approve[Auto-Approve Transaction]
    ScoreCheck -->|MRS 31-60: Monitor| Log[Log Profile for Batch Review]
    ScoreCheck -->|MRS 61-80: Suspicious| L1[Route to Level 1 Analyst Case Queue]
    ScoreCheck -->|MRS 81-100: High Risk| Freeze[Trigger Outgoing Fund Freeze + SMS Alert]

    Freeze --> L2[Route to Senior Fraud Investigator Case Queue]
    L1 --> Review1{Legitimate Activity?}
    
    Review1 -->|Yes| Unflag[Whitelist Profile & Unfreeze]
    Review1 -->|No| Escalate[Escalate to Senior Investigator]
    
    Escalate --> L2
    L2 --> Copilot[Launch AI Fraud Copilot]
    Copilot --> Draft[Auto-Generate Case Summary & SHAP Explainers]
    Draft --> FinalReview{Investigator Action}
    
    FinalReview -->|False Positive| Unfreeze[Release Account & Log Feedback Loop]
    FinalReview -->|Confirmed Fraud| Confirm[Mark Account as Fraudulent & Save to Neo4j Graph]
    
    Confirm --> FIU[Draft & File STR with FIU-IND / Regulatory Reporting]
    Unfreeze --> Retrain[Feed Legitimate Profile back to Feature Store for Drift Correction]
```

---

## SECTION 4: Governance Layer Architecture

This layer manages feature registry tracking, population stability metrics, bias thresholds, and independent model validations.

```mermaid
graph LR
    subgraph Governance_Pipeline [Audit & Validation Pipeline]
        Registry[Feature Registry] --> Monitor[Drift Monitor - PSI Checks]
        Monitor --> Fairness[Bias Monitor - Demographic Parity]
        Fairness --> Validation[Independent Model Validator]
        Validation --> Lineage[Audit Trail Logger]
    end

    subgraph Core_ML_Engine [Core Engine]
        ML[Active Model Weights]
    end

    ML -.->|Predictive Outputs| Monitor
    Monitor -.->|Trigger Retraining| ML
    Lineage -->|Regulatory Compliance Reports| Auditor[Regulatory Audit Team]
```

---

## SECTION 5: Mule Network Discovery Engine Graph Schema

The network intelligence model organizes raw data into a Neo4j Graph schema to expose circular transfers and layering clusters.

```mermaid
classDiagram
    direction LR
    class Account {
        +String Account_Number
        +String Occupation
        +String Risk_Segment
        +String Vintage
        +Float Current_Balance
        +Float Betweenness_Centrality
        +Float PageRank_Score
    }

    class Customer {
        +String Customer_ID
        +String Name
        +String KYC_Status
        +String Primary_Phone
        +String Risk_Tier
    }

    class Transaction {
        +String Txn_ID
        +Float Amount
        +String Timestamp
        +String Channel (UPI/IMPS/ATM)
        +String Location
    }

    Customer "1" --> "many" Account : OWNS
    Account "1" --> "many" Transaction : OUTBOUND_TRANSFER
    Transaction "many" --> "1" Account : BENEFICIARY
```

* ** Louvain Community Detection**: Detects communities of accounts that frequently route money within dense sub-networks, signaling structured fraud syndicates.
* **Circle/Cycle Detection**: Identifies transactional paths where funds return to the sender account ($A \to B \to C \to A$) within a 24-hour window, bypassing traditional cash-flow surveillance.
