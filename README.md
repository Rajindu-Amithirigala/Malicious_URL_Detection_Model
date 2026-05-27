# Malicious URL Detection

A machine learning system that classifies URLs into four categories — **phishing**, **benign**, **defacement**, and **malware** — using hand-engineered lexical features extracted from raw URL strings and an ensemble of three classifiers.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Usage](#usage)
- [How It Works](#how-it-works)
- [Dataset](#dataset)
- [Credits & Citations](#credits--citations)

---

## Overview

This project approaches URL classification as a **multi-class supervised learning problem**. Since only raw URL strings and their labels are provided, all input features are engineered from the URL structure itself — no external lookups or DNS queries required.

Three models are trained and combined into a **soft-voting ensemble**:

| Model | Description |
|---|---|
| Random Forest | Ensemble of decision trees, robust to noise |
| XGBoost | Gradient boosted trees, strong on tabular data |
| Logistic Regression | Linear baseline, highly interpretable |

The final prediction is made by averaging the confidence scores (probabilities) of all three models and picking the class with the highest average — a technique called **soft voting**.

---

## Features

13 lexical features are extracted from each URL:

| Feature | Description |
|---|---|
| `url_length` | Total character length of the URL |
| `has_ip` | Whether the hostname is a raw IP address |
| `dot_count` | Number of dots in the full URL |
| `at_count` | Number of `@` symbols (browser redirection trick) |
| `hyphen_count` | Number of hyphens in the domain |
| `path_depth` | Number of directory levels in the path |
| `query_length` | Length of the query string |
| `digit_ratio` | Ratio of digits to characters in the hostname |
| `domain_length` | Length of the full netloc |
| `subdomain_count` | Number of subdomains |
| `file_extensions` | Whether the path ends in a suspicious extension (`.exe`, `.zip`, `.rar`, `.js`, `.php`) |
| `brand_copying` | Whether a known brand name appears in the domain but the domain is not the official one |
| `typosquat_score` | Similarity score between the domain and known brand names — catches typosquatting like `paypa1.com` |

---

## Project Structure

```
├── Malicious_URL_Detection_ML.ipynb   # Main notebook
├── url_detector.py                    # Standalone runnable script with menu
├── malicious_phish.csv                # Dataset (not included — see Dataset section)
├── voting_clf.pkl                     # Saved voting classifier (generated after training)
├── scaler.pkl                         # Saved StandardScaler (generated after training)
├── requirements.txt                   # Python dependencies
└── README.md
```

---

## Installation

**1. Clone the repository**
```bash
git clone https://github.com/Rajindu-Amithirigala/malicious-url-detection.git
cd malicious-url-detection
```

**2. Create a virtual environment (recommended)**
```bash
python -m venv env
source env/bin/activate        # macOS / Linux
env\Scripts\activate           # Windows
```

**3. Install dependencies**
```bash
pip install -r requirements.txt
```

## Usage

### Running the notebook

Open `Malicious_URL_Detection_ML.ipynb` in Jupyter and run all cells:

```
Kernel → Restart & Run All
```

### Running the standalone script

```bash
python url_detector.py
```

You will see a menu:

```
==============================
   Malicious URL Detector
==============================

Select an option:
1. Train and evaluate all 3 models separately
2. Train and save voting classifier
3. Predict URL
4. Exit
```

**Option 1** — Trains Random Forest, XGBoost, and Logistic Regression individually and prints accuracy, classification report, and confusion matrix for each. Nothing is saved — this is for performance analysis only.

**Option 2** — Trains the soft-voting ensemble of all three models, evaluates it, and saves `voting_clf.pkl` and `scaler.pkl` to disk. Run this once before using option 3.

**Option 3** — Loads the saved model and predicts the class of any URL you enter. Requires option 2 to have been run first.

### Example prediction output

```
Final Prediction
URL: http://paypa1-secure.tk/login/verify
PREDICTION: PHISHING
PROBABILITY: 83.47%
```

---

## How It Works

```
Raw URL string
      │
      ▼
extract_features()       ← lexical feature engineering
      │
      ▼
StandardScaler           ← normalise feature values
      │
      ▼
Voting Classifier
  ├── Random Forest      → probability scores
  ├── XGBoost            → probability scores
  └── Logistic Regression→ probability scores
      │
      ▼
Average probabilities (soft vote)
      │
      ▼
Predicted class: phishing / benign / defacement / malware
```

**Important:** The same `StandardScaler` fitted on training data is used at prediction time. A new scaler must never be created at inference — doing so would produce different scaling values and break the model's learned decision boundaries.

---

## Dataset

**Malicious URLs Dataset** by Siddharth Mandloi (sid321axn)

- **Source:** [https://www.kaggle.com/datasets/sid321axn/malicious-urls-dataset](https://www.kaggle.com/datasets/sid321axn/malicious-urls-dataset)
- **Size:** 651,191 URLs
- **Classes:**
  - Benign: 428,103 URLs
  - Defacement: 96,457 URLs
  - Phishing: 94,111 URLs
  - Malware: 32,520 URLs
- **Format:** CSV with two columns — `url` (raw URL string) and `type` (class label)

The dataset was curated from five sources:

| Source | Purpose |
|---|---|
| URL dataset (ISCX-URL-2016) | Benign, phishing, malware, and defacement URLs |
| Malware Domain Blacklist | Additional phishing and malware URLs |
| Faizan Git Repo | Additional benign URLs |
| PhishTank | Additional phishing URLs |
| PhishStorm | Additional phishing URLs |

---

## Credits & Citations

**Dataset**

> Mandloi, S. (2021). *Malicious URLs Dataset*. Kaggle.
> [https://www.kaggle.com/datasets/sid321axn/malicious-urls-dataset](https://www.kaggle.com/datasets/sid321axn/malicious-urls-dataset)

**Original research and methodology**

> Mandloi, S. (2021). *Detection of Malicious URLs using Lexical Features and Boosted Machine Learning Algorithms*. GitHub.
> [https://github.com/sid321axn/Detection_of_Malicious_URLs](https://github.com/sid321axn/Detection_of_Malicious_URLs)

**Libraries used**

- [scikit-learn](https://scikit-learn.org/) — machine learning models, preprocessing, and evaluation
- [XGBoost](https://xgboost.readthedocs.io/) — gradient boosted classifier
- [pandas](https://pandas.pydata.org/) — data manipulation
- [NumPy](https://numpy.org/) — numerical operations
- [matplotlib](https://matplotlib.org/) & [seaborn](https://seaborn.pydata.org/) — visualisation
- [joblib](https://joblib.readthedocs.io/) — model serialisation

---

## License

This project is for educational purposes. The dataset is subject to its own license as specified on Kaggle by the original author.
