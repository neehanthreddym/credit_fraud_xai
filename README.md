# Explainable Credit Risk & Fraud Signals Engine

A prototype decisioning system that detects credit risk and fraud signals by combining graph neural networks, deep learning, and large language models for explainable AI-driven financial intelligence.


## Problem Statement

This project tackles the challenge of **detecting fraudulent credit card transactions in real-time** by leveraging graph-based machine learning techniques that capture relationships between customers, merchants, devices, and transaction patterns. 

Given a new transaction along with historical context up to that point in time, the goal is to predict whether the transaction is fraudulent, addressing the significant difficulty posed by extreme class imbalance where only **1.7% of transactions are fraudulent**.

The business context requires careful optimization:
- **Missing fraudulent transactions (false negatives)** leads to direct financial losses
- **Incorrectly flagging legitimate transactions (false positives)** damages customer experience and trust

This work explores how combining **temporal, geospatial, and relational features** within a graph neural network framework can improve fraud detection performance compared to traditional tabular approaches.


## Why Graphs for Fraud Detection?

Traditional fraud detection models treat each transaction independently, using only the transaction's features (amount, merchant, location, etc.). But real-world fraud doesn't happen in isolation—it happens through **networks of connected entities**.

### Think of it this way:
- A fraudster doesn't just make one transaction. They might use the same stolen card across multiple merchants or devices.
- Fraudulent merchants often appear in clusters, connected through similar customer profiles or geographic regions.
- A single compromised device might be linked to dozens of fraudulent transactions across different accounts.

**Graphs capture these connections.** By representing transactions as nodes and their relationships (same card, same device, same merchant, similar location) as edges, we can:
- **Spot fraud rings**: Groups of connected fraudulent entities that would look normal in isolation
- **Detect velocity patterns**: The same card being used in multiple locations within minutes
- **Leverage network effects**: If entity A is fraudulent and connected to entity B, entity B has a higher fraud risk
- **Understand context**: A $500 transaction might be normal for one customer but highly suspicious for another with different historical patterns

Graph Neural Networks (GNNs) learn from both the transaction features **and** the network structure, making them particularly powerful for fraud detection where relationships matter as much as individual attributes.


## Dataset Overview

**NeerajCodz creditCardFraudDetection** dataset:
- **Total transactions**: 1,856,000 (1.3M training, 556K test)
- **Features**: 23 raw features across 4 categories
- **Target variable**: `is_fraud` (binary: 0 = legitimate, 1 = fraudulent)
- **Class distribution**: 
  - Legitimate: 1,277,961 (98.31%)
  - Fraudulent: 22,039 (1.69%)

### Feature Categories
1. **Transaction Core**: Amount, category, merchant name, card number, timestamp
2. **Geolocation**: Customer/merchant coordinates (lat/long), address, city population density
3. **Customer Demographics**: Name, gender, date of birth, occupation
4. **Temporal**: Unix timestamp, transaction sequence number

### Device Inference
Device identification is inferred from credit card number (`cc_num`) patterns, enabling detection of:
- Same card used across multiple geographic locations (velocity fraud)
- Card reuse in high-fraud merchant categories
- Device linking to prior fraud cases through card number grouping


## Temporal Validity & Leakage Prevention

To ensure realistic fraud detection and prevent data leakage, enforce strict temporal constraints:

### Core Rules
- **Prediction unit**: One transaction
- **Label**: `is_fraud` (binary classification)
- **Temporal rule**: Only edges and features from **before** the transaction timestamp are allowed

### Implementation Guardrails
✅ **No future transactions are used**  
All features, graph edges, and aggregated statistics are computed using only transactions that occurred strictly before the current transaction's timestamp.

✅ **Graph snapshots are time-aware**  
Graph structures (customer-merchant edges, device-card linkages, etc.) are built incrementally. For each transaction at time `t`, use only the graph state as of time `t-1`.

✅ **Fraud history is aggregated strictly**  
Features like "number of fraudulent transactions by this merchant" or "fraud rate for this device" are computed using only historical data up to—but not including—the current transaction.

### Why This Matters
Without temporal safeguards, models can "cheat" by learning from future information that wouldn't be available at prediction time. This approach ensures that:
- Model performance metrics reflect real-world deployment scenarios
- The system can be safely deployed for real-time fraud detection
- Evaluation is honest and reproducible


## Architecture

### Data Layer
- **Tabular**: Credit/transaction style dataset (Parquet for analytics, MongoDB for events)
- **Graph**: Entity graph modeling customers, merchants, devices, and their relationships
- **Storage**: Parquet for analytics pipeline, MongoDB for model outputs and explanations

### Modeling Pipeline
Multiple model families for comprehensive comparison:

1. **Baseline**: Logistic Regression / GLM
2. **Tree-based**: XGBoost / Random Forest
3. **Deep Learning**:
   - Graph Neural Network (GCN or GraphSAGE) for relational risk modeling
   - LSTM for transaction sequences (optional temporal component)

### Evaluation Metrics
Critical business-focused evaluation:
- **Performance**: Accuracy, Precision, Recall, AUC-ROC
- **Financial Impact**: Expected Loss, Cost curves, ROI analysis at different thresholds
- **Fairness**: Model performance across demographic groups

### Explainability + GenAI
- **SHAP**: Feature-level contribution analysis for individual predictions
- **Graph Path Explanations**: Why two entities are linked and how that affects risk scoring
- **LLM-Generated Insights**:
  - Analyst-facing summaries: "This customer was flagged due to X, Y, Z patterns"
  - Auto-generated model documentation and evaluation reports


## Deliverables

[✔️] GitHub repo with clean structure

[✔️] README written like an internal research prototype

[ ] Technical notebook showing experiments and model comparisons

[ ] Business impact document with ROI analysis

[ ] Optional: FastAPI endpoint for real-time scoring + explanation


## License

This is a research prototype for demonstration purposes.