# 🛡️ Explainable AI for Network Intrusion Detection

An end-to-end machine learning pipeline for classifying network traffic into **5 attack categories** (Normal, DoS, Probe, R2L, U2R) using the NSL-KDD benchmark dataset, with full model explainability via SHAP.

---

## 📋 Project Overview

Network Intrusion Detection Systems (NIDS) are critical infrastructure in cybersecurity — but most ML-based detectors operate as black boxes. Security analysts can't act on alerts they don't understand.

This project builds a **transparent, explainable** intrusion detection system that not only classifies network connections with high accuracy but also provides **per-prediction explanations** of *why* a connection was flagged, using SHAP (SHapley Additive exPlanations).

### Key Results

| Metric | Random Forest (Baseline) | XGBoost (Optimized) | Improvement |
|--------|--------------------------|---------------------|-------------|
| **Accuracy** | 0.7403 | 0.7893 | +4.9% |
| **Macro F1** | 0.5234 | 0.6156 | +17.6% relative |
| **Weighted F1** | 0.6930 | 0.7549 | +6.2% |

### Per-Class Recall Improvement

| Attack Class | Baseline | XGBoost | Change |
|-------------|----------|---------|--------|
| DoS | 77.1% | 83.8% | ↑ +6.7% |
| Normal | 97.1% | 97.2% | ↑ +0.1% |
| Probe | 61.3% | 73.9% | ↑ +12.6% |
| R2L | 0.5% | 10.3% | ↑ +9.8% |
| U2R | 17.9% | 29.9% | ↑ +11.9% |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    DATA INGESTION                            │
│  NSL-KDD Dataset (125,973 train / 22,544 test)              │
│  41 features · 23 attack types · 5 categories               │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                EXPLORATORY DATA ANALYSIS                     │
│  Class distribution · Feature correlations · Statistical     │
│  profiling · Normal vs Attack distribution analysis          │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                 FEATURE ENGINEERING                           │
│  16 new features: traffic ratios, error composites,          │
│  log transforms, behavioral flags, interaction terms         │
│  Final: 57 features                                          │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              PREPROCESSING PIPELINE                          │
│  Label encoding (categoricals) · StandardScaler (fitted on   │
│  train only) · SMOTE oversampling (R2L: 995→8K, U2R: 52→4K) │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    MODELING                                   │
│  Baseline: Random Forest (200 trees, balanced weights)       │
│  Final: XGBoost + Optuna (40-trial Bayesian HPO)             │
│  Optimized for Macro F1 via 3-fold stratified CV             │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                 EXPLAINABILITY (SHAP)                         │
│  Global: Feature importance bar + beeswarm plots (all 5      │
│  classes) · Local: Waterfall plots for individual predictions │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔬 Technical Details

### Dataset: NSL-KDD
The NSL-KDD dataset is the de facto benchmark for evaluating network intrusion detection systems. It is a cleaned and improved version of the original KDD Cup '99 dataset, addressing issues of redundant records and class imbalance.

**5 Attack Categories:**
- **Normal** — Legitimate network traffic
- **DoS** (Denial of Service) — Flood attacks that overwhelm a target (e.g., neptune, smurf)
- **Probe** — Surveillance and port scanning (e.g., satan, ipsweep, nmap)
- **R2L** (Remote-to-Local) — Unauthorized remote access (e.g., guess_passwd, ftp_write)
- **U2R** (User-to-Root) — Privilege escalation (e.g., buffer_overflow, rootkit)

### Feature Engineering (16 new features)
| Category | Features | Rationale |
|----------|----------|-----------|
| Traffic Volume | `total_bytes`, `src_dst_byte_ratio`, `log_src_bytes`, `log_dst_bytes`, `log_total_bytes` | DoS floods show extreme send/receive imbalance |
| Error Composites | `serror_composite`, `rerror_composite` | Consolidates 4 highly correlated (r>0.95) error rate features into single scores |
| Behavioral Flags | `has_serror`, `has_rerror`, `zero_src_bytes`, `zero_dst_bytes`, `has_compromise_indicators` | Binary indicators of suspicious patterns |
| Interaction Terms | `same_srv_rate_x_count`, `dst_host_srv_diversity`, `dst_host_concentration`, `srv_count_ratio` | Captures non-linear feature relationships |

### Class Imbalance Handling
SMOTE (Synthetic Minority Oversampling Technique) was applied to training data only:
- R2L: 995 → 8,000 samples (+7,005 synthetic)
- U2R: 52 → 4,000 samples (+3,948 synthetic)

### Hyperparameter Optimization
Optuna Bayesian optimization (40 trials) over 9 hyperparameters, optimizing Macro F1 via 3-fold stratified cross-validation. Best CV score: **0.9986**.

### SHAP Explainability Insights
- **DoS detection** driven by: `dst_host_srv_count`, `count`, connection error rates
- **Probe detection** driven by: `total_bytes`, `dst_host_diff_srv_rate`, service diversity
- **R2L detection** driven by: `count`, `service`, `logged_in`, `dst_host_same_src_port_rate`
- **U2R detection** driven by: `dst_host_srv_count`, `duration`, `logged_in`, `num_file_creations`

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.10+ |
| Data Manipulation | Pandas, NumPy |
| Visualization | Matplotlib, Seaborn |
| ML Framework | Scikit-Learn, XGBoost |
| Imbalanced Learning | imbalanced-learn (SMOTE) |
| Hyperparameter Tuning | Optuna (Bayesian TPE) |
| Explainability | SHAP (TreeExplainer) |
| Environment | Google Colab / Jupyter Notebook |

---

## 🚀 Quick Start

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/explainable-ids.git
cd explainable-ids

# Install dependencies
pip install pandas numpy matplotlib seaborn scikit-learn xgboost shap optuna imbalanced-learn

# Run the notebook
jupyter notebook explainable_ids.ipynb
```

---

## 📁 Project Structure

```
explainable-ids/
├── explainable_ids.ipynb    # Full end-to-end notebook
├── README.md                # This file
├── requirements.txt         # Python dependencies
└── figures/                 # Saved visualizations
    ├── correlation_heatmap.png
    ├── class_distribution.png
    ├── confusion_matrix_comparison.png
    ├── shap_global_importance.png
    ├── shap_beeswarm_dos.png
    ├── shap_beeswarm_probe.png
    ├── shap_beeswarm_r2l.png
    ├── shap_beeswarm_u2r.png
    └── project_summary_dashboard.png
```

---

## 📝 Limitations & Future Work

- **R2L and U2R remain challenging** — These attack types intentionally mimic normal traffic patterns, making them inherently difficult to detect. This is a well-documented challenge in the NSL-KDD literature.
- **SMOTE limitations** — Synthetic oversampling for extremely small classes (U2R: 52 samples) may not capture the full diversity of real attack variants.
- **Future improvements:** Deep learning approaches (LSTM/Transformer for sequential connection patterns), ensemble stacking, and evaluation on the CICIDS2017 dataset for modern attack coverage.

---

## 📜 License

This project is open-source under the MIT License.

---

## 🙏 Acknowledgments

- [NSL-KDD Dataset](https://www.unb.ca/cic/datasets/nsl.html) — Canadian Institute for Cybersecurity
- [SHAP Library](https://github.com/shap/shap) — Lundberg & Lee, 2017
- [XGBoost](https://xgboost.readthedocs.io/) — Chen & Guestrin, 2016
- [Optuna](https://optuna.org/) — Akiba et al., 2019
