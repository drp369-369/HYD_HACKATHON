# MuleShield AI: Explainable Fraud Intelligence & Investigation Platform
### Round 1 Hackathon Submission Package
**Target Problem:** AI/ML-Based Classification of Suspicious Mule Accounts  
**Target Variable:** `F3924` (Mule Account Flag)  
**Dataset Reference:** `DataSet.csv` (9,082 rows, 3,925 columns)  

---

## 📌 Project Overview
**MuleShield AI** is an enterprise-grade **Explainable Fraud Intelligence & Investigation Platform** designed to proactively identify money mule accounts using banking transaction features. 

Unlike simple machine learning models, MuleShield AI integrates real-time behavioral modeling, graph-based network analysis (Neo4j), explainable AI (SHAP), automated regulatory compliance templates, and a generative AI assistant (AI Fraud Copilot) to support banking investigators and satisfy strict regulatory rules (such as RBI Guidelines, PMLA, and FATF recommendations).

---

## 📁 Repository Structure
* **[submission_package.md](file:///c:/Users/itzpa/Downloads/HYD_HACKATHON/submission_package.md)**: The core submission document consisting of 18 comprehensive sections, outlining executive vision, dataset audit, model validation metrics, risk engine math, and ROI projections.
* **[architecture_diagram.md](file:///c:/Users/itzpa/Downloads/HYD_HACKATHON/architecture_diagram.md)**: High-level architectural visual blueprints using Mermaid diagrams, mapping the 10-layer architecture, streaming sequences, case workflows, and graph schemas.
* **[pitch_presentation.md](file:///c:/Users/itzpa/Downloads/HYD_HACKATHON/pitch_presentation.md)**: A 15-slide presentation deck designed for judges, covering financial impacts, data leakage discoveries, competitive advantage, and roadmap.

---

## 🔍 Key Data Audit & Leakage Discoveries

During our programmatic audit of the `DataSet.csv`, we exposed two critical data leakage features that we have explicitly excluded from our modeling pipeline to prevent overfitting and ensure real-world deployability:

### 1. Feature `F3912` (Correlation Leakage)
* **Discovery**: Pearson correlation of **0.969066** with target. Matches target in 9,077 of 9,082 rows, differing in only 5 rows.
* **Operational Risk**: Identified as an administrative action flag (e.g., "Account Frozen") applied *after* the investigation completes. In production, this flag is not set at the time of monitoring. Including it results in 99.9% fake accuracy but catches 0% fraud in production.

### 2. Feature `F2230` (Reporting Period Leakage)
* **Discovery**: Perfect class separation based on the reporting month.
  * Every single normal account (9,001 rows) has `F2230` set to `'Oct25'`.
  * Every single mule account (81 rows) has `F2230` set to either `'Sep25'`, `'Nov25'`, or `'Dec25'`.
* **Operational Risk**: A synthetic data assembly artifact. Tree models split on month and achieve 100% fake validation accuracy, but collapse in production.

---

## 📊 True Machine Learning Performance

We ran a Stratified 5-Fold Cross Validation on the dataset, selecting the top 100 features dynamically per fold, and explicitly dropping both leakages to measure the true predictive signal of clean transaction features:

| Model Name | Precision | Recall | F1-Score | ROC-AUC | PR-AUC |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Logistic Regression** | 0.0477 | 0.6919 | 0.0891 | 0.8527 | 0.0950 |
| **Random Forest** | 0.8433 | 0.2346 | 0.3486 | 0.9297 | 0.4209 |
| **XGBoost (Recommended)** | **0.6960** | **0.3838** | **0.4758** | **0.9323** | **0.5147** |
| **LightGBM** | 0.4045 | 0.3338 | 0.3577 | 0.8739 | 0.3569 |
| **CatBoost** | **0.5670** | **0.4199** | **0.4665** | **0.9171** | **0.4588** |
| **Isolation Forest** | 0.0365 | 0.0368 | 0.0364 | 0.6407 | 0.0325 |

---

## 🛡️ Robust Security Integration

Fraud detection platforms are primary targets for sophisticated adversaries. MuleShield AI embeds enterprise-grade security at every layer of its architecture:

### 1. Data Protection & Privacy (PII Tokenization)
* **Risk**: Banking ledgers contain Personally Identifiable Information (PII) like names, phone numbers, and balances. Storing this in plaintext in the ML feature store violates privacy laws (e.g., DPDPA 2023 in India, GDPR in Europe).
* **Security Integration**: Implement **Format-Preserving Encryption (FPE)** and SHA-256 hashing on all PII columns (e.g., `Customer_ID`, `Phone_Number`). ML feature stores only ingest tokenized representations, preventing database exposure.

### 2. API Security & Rate-Limiting
* **Risk**: Fraudsters can query model endpoints repeatedly to map out the decision boundaries (Model Extraction Attack).
* **Security Integration**: Implement **mutual TLS (mTLS)** for all microservices communications. Secure external API gateways using OAuth 2.0 access tokens and strict rate-limiting (e.g., max 100 queries/minute per API client) to prevent extraction attempts.

### 3. Adversarial Machine Learning Defenses
* **Risk**: Fraudsters manipulate transaction patterns (e.g., split amounts, alternate transaction channels) to bypass model detection (Adversarial Evasion Attack).
* **Security Integration**: 
  * **Adversarial Training**: Augment the training data with simulated adversarial transaction patterns.
  * **Input Perturbation Defenses**: Smooth incoming transaction counts slightly using noise injection to prevent minor updates from altering model decisions.
  * **Dual Engine Validation**: Run the unsupervised **Isolation Forest** in parallel. If XGBoost predicts "Normal" but Isolation Forest flags it as a high-density anomaly, the transaction is routed to manual review.

### 4. Immutable Audit Trail (Model Lineage)
* **Risk**: Internal actors could alter model thresholds to whitelist specific mule accounts.
* **Security Integration**: Every model transaction, score change, and analyst override is logged to an immutable audit ledger. This ensures feature values and scores cannot be modified without generating an audit alert.

---

## 🚀 Future Roadmap & Optimizations

1. **Graph Neural Networks (GNNs)**: Implement Node2Vec and GraphSAGE models in Neo4j to capture the structural network relationships of transaction chains natively as node embeddings.
2. **Federated Learning**: Train model weights across multiple distinct banks without sharing raw transaction databases, establishing a shared cross-bank threat intelligence network.
3. **Low-Latency Feature Streaming**: Deploy Feast with Redis to serve real-time velocity features within a <50ms window.
