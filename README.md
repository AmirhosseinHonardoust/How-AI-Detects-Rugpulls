# **How AI Detects Rugpulls Before Humans Notice: Inside Token Risk Scoring**

### A Deep Dive into Feature Engineering, Security Patterns, and ML-Inspired Analysis for ERC-20 Token Scam Detection

---            
   
## Introduction   

Rugpulls are one of the most common and devastating attack patterns in the Web3 ecosystem.
A malicious token is deployed, marketed, pumped, and at the moment of maximum hype:

* the owner mints infinite tokens,
* or enables a 99% tax,
* or freezes trading,
* or blacklists all users,
* or drains liquidity

and the project collapses in minutes.

The challenge is **most of these risks are hidden inside the token’s Solidity code**, invisible to non-technical buyers, and sometimes even invisible to developers who don’t carefully read the contract.

This is where **AI-inspired, feature-based risk scoring** enters the scene.

Instead of reading the code manually, we can **teach Python to analyze Solidity**, extract behavioral signals, convert them into numeric features, and generate an interpretable risk score that reflects the likelihood of malicious behavior.

This article explains exactly how that works, step by step.

---

# 1. Rugpull Patterns Are Predictable

Despite creativity among scammers, **rugpull contracts share consistent structural behaviors**, such as:

### **1. Owner minting**

The owner can mint unlimited supply.

```solidity
function mint(address to, uint256 amount) external onlyOwner {
    _mint(to, amount);
}
```

### **2. Fee manipulation**

The owner can dynamically change tax rates.

```solidity
function setFee(uint256 newFee) external onlyOwner {
    fee = newFee;
}
```

### **3. Blacklists**

Restrict who can transfer tokens (or just block sellers).

```solidity
mapping(address => bool) isBlacklisted;
```

### **4. Trading control**

Owner decides when trading is open.

```solidity
bool tradingOpen = false;
```

### **5. maxTx or anti-whale mechanisms**

Often used as a hidden honeypot mechanism.

```solidity
uint256 maxTxAmount = totalSupply() / 100;
```

These patterns are **detectable by simple analysis**, without needing complex blockchain simulation.

---

# 2. Turning Solidity into Machine-Readable Signals

AI cannot “understand” Solidity code directly.
So we convert smart-contract text into **structured features**, just like transforming raw text into structured NLP signals.

### Example feature vector:

|      Feature       | Value |
| ------------------ | ----- |
|     `n_lines`      |  143  |
|     `n_public`     |   6   |
|    `n_external`    |   1   |
|  `has_owner_mint`  |   1   |
|    `has_set_fee`   |   1   |
|   `has_blacklist`  |   0   |
| `has_trading_lock` |   1   |
|    `has_max_tx`    |   0   |

This numeric representation allows scoring, modeling, and ML.

---

# 3. Extracting Features with Pure Python

A feature extractor scans text for patterns:

```python
DANGEROUS_PATTERNS = {
    "has_mint": r"\bmint\s*\(",
    "has_owner_mint": r"onlyOwner[\s\S]*function\s+mint",
    "has_set_fee": r"setFee|setTax|setBuyFee|setSellFee",
    "has_blacklist": r"blacklist|isBlacklisted",
    "has_trading_lock": r"tradingOpen|enableTrading|disableTrading|lockTrading",
    "has_max_tx": r"maxTxAmount|maxTransactionAmount|maxTx"
}
```

Each pattern becomes a feature (\in {0,1}).

This is equivalent to **binary NLP features** commonly used in classical ML models.

---

# 4. Risk Scoring, The ML-Inspired Engine

The scoring model is **not random**.
It’s based on real-world rugpull mechanics and weighted like an ML feature importance map.

### Core idea:

> Risk = weighted sum of dangerous features.

### Example weights:

* Owner minting → **+40**
* Fee manipulation → **+25**
* Trading lock → **+25**
* Blacklisting → **+20**
* maxTx → **+15**
* Complex contract (>800 lines) → **+15**

### Risk Score Equation

<img width="213" height="74" alt="Screenshot 2025-11-17 at 17-45-41 Repo style analysis" src="https://github.com/user-attachments/assets/6ec84660-d270-45a5-a74f-449c70bea69a" />

Where:

<img width="229" height="64" alt="Screenshot 2025-11-17 at 17-47-13 Repo style analysis" src="https://github.com/user-attachments/assets/dd75238e-c7ac-4186-987b-ce547064a69a" />

### Example:

A contract with owner mint + fee control + blacklist:

[
40 + 25 + 20 = 85
]

→ **High Risk**
→ **rugpull_candidate**

---

# 5. Risk Levels & Labels

The final score maps to qualitative categories:

| Range  | Level  | Label             |
| ------ | ------ | ----------------- |
| 0–20   | Low    | safe              |
| 21–60  | Medium | suspicious        |
| 61–100 | High   | rugpull_candidate |

These categories mirror real-world auditor language:
*“safe”, “needs review”, “dangerous”.*

---

# 6. Example Walkthroughs

## Example: Safe Token

Features:

```json
{
  "has_mint": 0,
  "has_owner_mint": 0,
  "has_blacklist": 0,
  "has_trading_lock": 0,
  "has_set_fee": 0
}
```

Score:

```
0
```

Label:

```
safe
```

---

## Example: Suspicious Token

Features:

```json
{
  "has_max_tx": 1,
  "has_trading_lock": 1
}
```

Score:

```
15 + 25 = 40
```

Label:

```
suspicious
```

Often these contracts become honeypots **after deployment**.

---

## Example: Rugpull Candidate

Features:

```json
{
  "has_owner_mint": 1,
  "has_set_fee": 1,
  "has_blacklist": 1,
  "has_trading_lock": 1
}
```

Score:

```
40 + 25 + 20 + 25 = 110 → capped at 100
```

Label:

```
rugpull_candidate
```

This matches thousands of real scam patterns.

---

# 7. Why AI Can Detect Rugpulls Better Than Humans

Humans:

* Overlook small functions
* Do not scan for patterns systematically
* Cannot compare across thousands of contracts
* Cannot assign consistent numeric weight to features

AI / ML-inspired models:

Never forget patterns
Execute rules with perfect consistency
Evaluate contracts in milliseconds
Produce interpretable numeric scores
Scale across thousands of tokens

This transforms security analysis from **subjective interpretation**
→ into **repeatable, measurable risk computation**.

---

# 8. How This Becomes a Real ML Model

The current scoring system mimics ML structure:

* Features → numeric vector
* Weighted sum → prediction
* Label → classification

To add real ML:

### Step 1: Build dataset

For each contract:

* Extract features
* Assign real labels (`scam`, `legit`)
* Store as rows in a CSV

### Step 2: Train ML model

```python
from sklearn.ensemble import RandomForestClassifier
clf = RandomForestClassifier()
clf.fit(X, y)
```

### Step 3: Replace heuristic model

Use `clf.predict()` and `clf.predict_proba()` instead.

### Step 4: Add explainability

Use SHAP to show:

* Which features drive risk
* Which functions are dangerous
* Why the model flagged a token

---

# 9. Limitations of Static AI Auditing

AI-based static analysis is powerful, but has limits:

* Cannot detect runtime exploits
* Cannot detect liquidity behavior
* Cannot detect MEV-related manipulation
* Cannot detect upgradeable proxy traps
* Cannot detect multi-contract interactions

However, for **ERC-20 token scams**, static analysis captures **80%+** of common rugpull patterns.

---

# 10. Conclusion

AI doesn’t need deep learning to detect rugpulls,
it only needs:

* smart feature engineering,
* domain knowledge,
* weighted scoring logic,
* and a clean data pipeline.

By transforming Solidity code into structured numeric signals, we can build systems that:

* warn investors
* assist auditors
* analyze thousands of tokens per day
* help exchanges filter dangerous contracts
* power automated tools such as block explorers and wallets

The future of Web3 security will be **hybrid**:

> Human understanding + AI consistency
>
> Security expertise + machine-scale analysis
>
> Pattern detection + interpretability

And tools like the **ML-Powered Token Launch Auditor** are the first step.
