# ai-py-aml
# AI & Python for AML  
*A practical reference for Product Owners & Lead Analysts*

---

## 0. Setup

```python
# Basic imports you may reuse
import pandas as pd
import numpy as np
```

---

## 1. Transaction Monitoring (TM)

**What it is:** Systems that detect suspicious transactions using rules or models.

**Key items to remember:**
- **Threshold rules:** e.g., daily total > 10,000  
- **Velocity rules:** many small transactions in short time  
- **Behavioral rules:** deviation from customer’s normal pattern  
- **False positives:** major pain point  
- **Explainability:** required by regulators  

### 1.1 Threshold rule example

```python
import pandas as pd

df = pd.DataFrame({
    "customer_id": [1,1,2,2,3],
    "amount": [5000, 6000, 2000, 9000, 15000],
    "date": ["2024-01-01"]*5
})

daily_totals = df.groupby(["customer_id", "date"])["amount"].sum().reset_index()
alerts = daily_totals[daily_totals["amount"] > 10000]

alerts
```

**Q:** Why do TM systems generate many false positives?  
**A:** Because rules are rigid and don’t understand context (e.g., legitimate high‑volume businesses look suspicious).

---

## 2. Structuring (Smurfing)

**What it is:** Breaking large cash deposits into smaller ones to avoid reporting thresholds.

**Key items:**
- **Cash deposits** just below threshold  
- **Repeated amounts** like 9,900  
- **Short time window** (e.g., 24–72 hours)  
- **Multiple locations** sometimes used  

### 2.1 Simple structuring detection

```python
df = pd.DataFrame({
    "customer_id": [1,1,1,2,2,3],
    "amount": [9900, 9800, 9700, 2000, 3000, 9500],
    "type": ["cash", "cash", "cash", "cash", "cash", "cash"],
    "date": ["2024-01-01"]*6
})

cash_tx = df[(df["type"] == "cash") & (df["amount"] < 10000)]
counts = cash_tx.groupby(["customer_id", "date"]).size().reset_index(name="count")

alerts = counts[counts["count"] >= 3]
alerts
```

**Q:** Simplest way to detect structuring?  
**A:** Count sub‑threshold cash deposits within a time window.

---

## 3. KYC / CDD / EDD Risk Scoring

**What it is:** Assigning a risk score to customers at onboarding and periodically.

**Key items:**
- **High‑risk country**  
- **Cash‑intensive business**  
- **No prior banking history**  
- **PEP (Politically Exposed Person)**  
- **Sanctions exposure**  

### 3.1 Simple risk scoring function

```python
def kyc_risk_score(country_risk, cash_business, no_history, pep):
    score = 0
    score += 40 if country_risk else 0
    score += 30 if cash_business else 0
    score += 20 if no_history else 0
    score += 50 if pep else 0
    return score

customers = [
    {"id": 1, "country_risk": True,  "cash_business": False, "no_history": True,  "pep": False},
    {"id": 2, "country_risk": False, "cash_business": True,  "no_history": False, "pep": True},
]

for c in customers:
    c["risk_score"] = kyc_risk_score(c["country_risk"], c["cash_business"], c["no_history"], c["pep"])

customers
```

**Q:** Why is KYC risk scoring important?  
**A:** It drives monitoring intensity and investigation priority.

---

## 4. Sanctions Screening (Name Matching)

**What it is:** Matching customer names against sanctions lists (OFAC, UN, EU, etc.).

**Key items:**
- **Fuzzy matching** needed (spelling, transliteration)  
- **False positives** common  
- **Aliases** and multiple spellings  

### 4.1 Fuzzy name matching example

```python
!pip install fuzzywuzzy[speedup] python-levenshtein -q
from fuzzywuzzy import fuzz

name = "Mohamed Al-Sayed"
sanctioned = ["Muhammad Alsayed", "John Doe", "Mohamad Alsayed"]

for s in sanctioned:
    score = fuzz.token_sort_ratio(name, s)
    print(f"{name} vs {s} -> similarity: {score}")
```

**Q:** Why is sanctions screening hard?  
**A:** Names vary across languages, spellings, and formats.

---

## 5. Anomaly Detection (Unsupervised ML)

**What it is:** ML that finds unusual patterns without predefined rules.

**Key items:**
- **Unsupervised ML** (Isolation Forest, Autoencoders)  
- Detects **outliers**  
- Helps reduce false positives  
- **Explainability** is challenging  

### 5.1 Isolation Forest example

```python
!pip install scikit-learn -q
from sklearn.ensemble import IsolationForest
import numpy as np
import pandas as pd

amounts = np.array([[50], [60], [55], [52], [5000], [48], [53], [49], [51]])
model = IsolationForest(contamination=0.1, random_state=42)
model.fit(amounts)

pred = model.predict(amounts)  # -1 = anomaly, 1 = normal
df = pd.DataFrame({"amount": amounts.flatten(), "prediction": pred})
df
```

**Q:** Biggest challenge with anomaly detection in AML?  
**A:** Explaining why something is anomalous to regulators and investigators.

---

## 6. Alert Scoring / Prioritization

**What it is:** ML model ranks alerts by risk so investigators focus on the most important ones.

**Key items:**
- Uses **historical SARs** and outcomes  
- Uses **customer behavior** and **context**  
- Must be **explainable**  
- Often used to **prioritize**, not auto‑close  

### 6.1 Simple heuristic scoring

```python
alerts = pd.DataFrame({
    "alert_id": [1,2,3],
    "high_risk_country": [1,0,1],
    "sudden_spike": [1,1,0],
    "many_counterparties": [0,1,1]
})

def alert_score(row):
    return (
        50 * row["high_risk_country"] +
        30 * row["sudden_spike"] +
        20 * row["many_counterparties"]
    )

alerts["score"] = alerts.apply(alert_score, axis=1)
alerts.sort_values("score", ascending=False)
```

**Q:** Why do banks use alert scoring?  
**A:** To reduce backlog and focus on high‑risk alerts first.

---

## 7. Network / Graph Analysis

**What it is:** Detecting relationships between accounts to find mule networks and layering.

**Key items:**
- **Nodes = accounts / entities**  
- **Edges = transactions / relationships**  
- Detects **rings**, **clusters**, **hubs**  
- Great for **money mule** and **fraud rings**  

### 7.1 Simple graph example

```python
!pip install networkx -q
import networkx as nx

G = nx.Graph()

edges = [
    ("A", "B"),
    ("B", "C"),
    ("C", "D"),
    ("A", "E"),
    ("E", "F"),
    ("X", "Y")
]

G.add_edges_from(edges)

list(nx.connected_components(G))
```

**Q:** Why is graph analysis powerful in AML?  
**A:** Criminals rarely act alone; networks reveal hidden connections.

---

## 8. NLP for AML Investigations

**What it is:** AI that reads and interprets text (narratives, news, documents).

**Key items:**
- **Narrative classification** (e.g., structuring vs fraud)  
- **SAR drafting assistance**  
- **Adverse media screening**  
- **Entity extraction** (names, locations, amounts)  

### 8.1 Zero‑shot narrative classification (conceptual)

```python
# This requires transformers and a model; shown as a pattern.
!pip install transformers -q
from transformers import pipeline

classifier = pipeline("zero-shot-classification", model="facebook/bart-large-mnli")

text = "Customer made 12 cash deposits of $9,900 over 3 days."
labels = ["Structuring", "Money Mule", "Fraud", "Sanctions Evasion"]

result = classifier(text, labels)
result
```

**Q:** Biggest NLP use case in AML?  
**A:** Classifying suspicious activity narratives and summarizing cases.

---

## 9. Adverse Media Screening

**What it is:** Scanning news for negative information about customers.

**Key items:**
- Uses **NLP** to detect crime, corruption, sanctions, etc.  
- Supports **EDD** (Enhanced Due Diligence)  
- Helps identify risk before transactions  

**Conceptual example:**  
- Input: “John Doe arrested for corruption in 2023.”  
- Output: Risk tag: **Corruption**, **High risk**, **EDD required**.

**Q:** Why is adverse media important?  
**A:** It reveals risk that may not appear in internal data.

---

## 10. Case Management Workflow

**What it is:** How alerts become investigations and possibly SARs.

**Key items:**
- **Alert → Case → Investigation → SAR**  
- Needs **audit trail**  
- Needs **evidence collection**  
- Needs **narrative writing**  

### 10.1 Simple case structure (Python dict)

```python
case = {
    "case_id": 123,
    "customer_id": 1,
    "alerts": [101, 102],
    "transactions": [5000, 6000, 9900],
    "status": "Under Review",
    "investigator_notes": []
}

case
```

**Q:** Most time‑consuming part of investigations?  
**A:** Reviewing transactions and writing SAR narratives.

---

## 11. Explainability

**What it is:** Ability to explain why a rule or model triggered an alert.

**Key items:**
- Required by **regulators**  
- Must be **human‑readable**  
- Applies to **rules** and **ML models**  
- “Black box” models are risky  

### 11.1 Simple explanation example

```python
def explain_structuring(transactions):
    explanation = []
    if len(transactions) >= 3 and all(a < 10000 for a in transactions):
        explanation.append("Multiple sub-threshold cash deposits detected.")
    return explanation

explain_structuring([9900, 9800, 9700])
```

**Q:** Why is explainability critical?  
**A:** Regulators and auditors must understand the logic behind detection.

---

## 12. Data Pipelines in AML

**What it is:** How data flows into AML systems.

**Key items:**
- **Customer data** (KYC)  
- **Transaction data**  
- **External lists** (sanctions, PEP, media)  
- **Batch + real‑time** ingestion  
- **Data quality** is everything  

### 12.1 Simple data quality check

```python
df = pd.DataFrame({
    "customer_id": [1,2,3],
    "country": ["US", None, "DE"],
    "occupation": ["Engineer", "Unknown", None]
})

missing = df.isnull().sum()
missing
```

**Q:** #1 cause of AML system issues?  
**A:** Poor data quality.

---

## 13. SAR (Suspicious Activity Report)

**What it is:** Regulatory report filed when suspicious activity is confirmed.

**Key items:**
- Filed within **regulatory deadlines**  
- Includes **narrative** and **evidence**  
- Bank **cannot notify** the customer  
- Must be **clear, factual, concise**  

### 13.1 Simple SAR narrative template

```python
def sar_narrative(customer_id, pattern, amounts, dates):
    return (
        f"Customer {customer_id} exhibited {pattern} behavior. "
        f"Transactions totaling {sum(amounts)} occurred on {', '.join(dates)}. "
        f"Activity appears inconsistent with known customer profile."
    )

sar_narrative(1, "structuring", [9900, 9800, 9700], ["2024-01-01", "2024-01-02", "2024-01-03"])
```

**Q:** Hardest part of SAR writing?  
**A:** Telling a clear story with evidence and no speculation.

---

## 14. Model Validation

**What it is:** Independent review of AML models.

**Key items:**
- **Data validation**  
- **Performance metrics** (precision, recall)  
- **Threshold tuning**  
- **Backtesting**  
- **Documentation**  

### 14.1 Simple backtest idea (conceptual)

```python
# Suppose we have labels: 1 = suspicious, 0 = not suspicious
y_true = np.array([1,0,1,0,1])
y_pred = np.array([1,0,0,0,1])

from sklearn.metrics import classification_report
print(classification_report(y_true, y_pred))
```

**Q:** Why is validation required?  
**A:** To prove the model works, is stable, and is not biased.
