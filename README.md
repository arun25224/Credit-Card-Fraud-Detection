# Credit Card Fraud Detection

# **Project Overview**

Built an interpretable, rule-based fraud detection system using a Classification and Regression Tree (CART). Instead of optimizing purely for accuracy, this project prioritizes **model transparency** and **operational actionability**, translating machine learning outputs into clear `IF-THEN` business rules for fraud analysts.

---

# **Problem Statement & Business Context**

Credit card fraud results in billions in annual losses, but traditional "black-box" models (like deep neural networks or complex ensembles) are often rejected by compliance teams due to lack of transparency.

**Business Goal:** Deliver a highly transparent model that catches **100% of fraudulent transactions** (maximize recall) while providing clear, auditable decision rules that human analysts can use to triage alerts efficiently.

---

# **Dataset & Preprocessing**

- **Source:** 10,000 synthetic transactions dataset
- **Features:** `amount`, `transaction_hour`, `merchant_category`, `foreign_transaction`, `location_mismatch`, `device_trust_score`, `velocity_last_24h`, `cardholder_age`
- **Target:** `is_fraud` (Binary: 0 = Legitimate, 1 = Fraud)
- **Preprocessing Steps:**
    - Dropped `transaction_id` (no predictive signal, potential data leakage risk)
    - One-Hot Encoded `merchant_category` to handle categorical data
    - Applied stratified train/test split (`test_size=0.2`) to preserve class distribution
    - Used `class_weight='balanced'` to intrinsically handle class imbalance without oversampling

---

# **Methodology & Thought Process**

### **1. Why CART over complex models?**

Fraud operations require **explainability**. A shallow decision tree (`max_depth=4`) produces easily readable decision paths. I prioritized interpretability over marginal accuracy gains, knowing that fraud analysts need clear rules, not just probabilities.

<img width="2048" height="1026" alt="image" src="https://github.com/user-attachments/assets/35a5716c-602b-42a9-a47e-6b7857aa9d86" />

### **2. Handling Imbalance & Overfitting**

Fraud datasets are heavily skewed. Instead of Synthetic Minority Over-sampling Technique (SMOTE)/undersampling (which can introduce synthetic noise), my group and I used `class_weight='balanced'` to penalize misclassification of the minority class. `max_depth=4` was deliberately chosen to prevent overfitting while keeping the tree shallow enough to extract ≤3 clear business rules.

### **3. Metric Selection Strategy**

- **Accuracy:** Misleading on imbalanced data (92%, but masks poor minority class performance).
- **Recall:** **Primary metric.** In fraud, missing a fraud case (False Negative) is costlier than reviewing a false alarm.
- **Precision:** Expected to be lower. Acceptable trade-off since manual review queues can handle a 16% false positive rate.
- **AUC-ROC:** Measures ranking quality across thresholds. 0.9878 indicates excellent separability.

<img width="989" height="590" alt="image" src="https://github.com/user-attachments/assets/3a9bfd58-cbbd-4671-b8b3-f0cdb969446c" />


### **4. Business Translation**

Machine learning models are useless if they don't drive action. I programmatically extracted the highest-fraud-probability leaf nodes from the trained tree and translated them into plain-English `IF-THEN` rules that can be deployed directly into rule engines or SOPs.

---

# **Results & Model Evaluation**

| **Metric** | **Logistic Regression** | **CART** | **Which is Better?** | **Why does this matter?** |
| --- | --- | --- | --- | --- |
| **Accuracy** | 0.9917 | 0.9240 | Logistic Regression | Measures the overall percentage of correct predictions out of all predictions made. |
| **Precision** | 0.8125 | 0.1648 | Logistic Regression | Focuses on the quality of the positive predictions |
| **Recall** | 0.5778 | 1.0000 | CART | Evaluates the model's completeness in catching actual fraud |
| **F1-Score** | 0.6753 | 0.2830 | Logistic Regression | Combines both Precision and Recall into a single, balanced metric |
| **AUC-ROC** (Area Under the Receiver Operating Characteristic curve) | 0.9852 | 0.9878 | CART | Indicates the overall ability to distinguish between legitimate and fraudulent classes across all thresholds. |

---

# **Top 3 Business Scenarios (100% Guaranteed Fraud)**

The model's decision paths were translated into operational scenarios:

| **Scenario** | **Description** | **Time** | **Origin** | **Amount** | **Action** |
| --- | --- | --- | --- | --- | --- |
| **1**
 | Off-Peak International Vulnerability | Before 3.30 AM | International | More than $7.02 | Immediate Automated Block |
| **2** | Off-Peak Domestic Discrepancy | Before 3.30 AM | Domestic | - | Flag as absolutely fraudulent as domestic origin does not guarantee safety |
| **3** | Standard-hour Geolocation Manipulation | After 3.30 AM | International | - | Strict Monitoring and blocking. Activity is masked during normal hours. |

**Note:** Amount matters for Scenario 1 as they are some legitimate transactions along with fraudulent ones. Amount does not matter for the other 2 scenarios as all transactions are fraudulent.

---

# **Deployment Strategy: Hybrid Approach**

To balance security rigor with customer experience, we recommended a hybrid deployment model rather than relying on a single algorithm for all decisions.

- **Avoid Standalone CART Deployment:** Using the CART model as a strict, standalone hard-blocking tool is ill-advised. Due to its sensitivity, relying on it exclusively would likely be catastrophic for the customer experience, leading to mass false declines of legitimate transactions.
- **Safe Live-Scoring with Logistic Regression (LR):** Logistic Regression is identified as the safer choice for baseline automated live-scoring. Its probabilistic nature allows for nuanced decision-making that protects the customer experience by minimizing false positives while still flagging risk.
- **The Hybrid Solution:** The optimal strategy is to deploy Logistic Regression for general automated decisions. Simultaneously, we should hardcode the CART model's highly specific "100% probability" scenarios as immediate hard-block triggers. This integrates the specific, high-confidence rules of CART within the broader, more flexible security architecture of Logistic Regression.

# **Key Takeaways & Skills Developed**

### **Technical Skills**

- **Python & Data Stack:** Pandas, Scikit-Learn, Matplotlib, NumPy
- **Modeling & Tuning:** Decision Trees (CART), class weight balancing, depth constraint regularization, stratified splitting
- **Evaluation & Metrics:** Precision-Recall trade-off analysis, ROC-AUC interpretation, confusion matrix reasoning
- **Visualization:** Custom tree plotting with class-based coloring, feature importance bar charts
- **Rule Extraction:** Programmatic traversal of decision trees to generate human-readable business logic

### **Analytical & Business Skills**

- **Problem Framing:** Aligned ML objectives with real-world fraud operations (Recall > Accuracy)
- **Imbalanced Data Strategy:** Chose algorithmic weighting over synthetic sampling for transparency and stability
- **Model-to-Business Translation:** Bridged technical outputs to operational SOPs using `IF-THEN` rule mapping
- **Metric-Driven Decision Making:** Justified precision trade-offs using business cost-benefit logic
- End-to-end ML pipeline execution (raw CSV → production-ready rules)
- Prioritizing interpretability and compliance over marginal accuracy gains

---

# **Next Steps & Future Improvements**

- Implement threshold tuning to optimize Precision-Recall curve for specific business cost ratios
- Compare with `RandomForest` & `XGBoost` for maintainable interpretability
- Deploy rules into a lightweight API (FastAPI) for real-time transaction scoring
- Add temporal features & velocity smoothing for dynamic risk scoring
