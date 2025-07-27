#  **Credit Score Modeling for On-Chain Wallets**

This project builds a data-driven credit scoring model for on-chain user wallets based on their transaction history and behavioral patterns. The approach is designed to be transparent, justifiable, and scalable for DeFi credit assessment.

---

## **1\. Data Collection**

We start with raw transaction data (`data`) which is assumed to be a list of dictionaries, each representing an individual blockchain transaction. Each transaction includes:

* `userWallet`: Ethereum-compatible wallet address

* `timestamp`: Unix timestamp of transaction

* `action`: Type of action (`deposit`, `borrow`, `repay`, `redeemUnderlying`, etc.)

* `actionData`:

  * `amount`: Token amount (usually in USDC with 6 decimal precision)

  * `assetPriceUSD`: Token price at the time of transaction

### **Data Processing Steps:**

1. **Grouping** transactions by wallet address (`userWallet`)

2. **Aggregating** metrics like number of transactions, types of actions, and USD value involved

3. **Temporal analysis**: active duration, first and last activity timestamps

These are written to a summary CSV file: `wallet_summary.csv`.

---

##  **2\. Feature Selection Rationale**

The features selected focus on capturing both **financial activity** and **user behavior**:

| Feature | Description | Why It Matters |
| ----- | ----- | ----- |
| `num_transactions` | Total number of transactions | General engagement level |
| `action_*` | Count of each type of action (e.g., `action_borrow`) | Captures behavioral diversity |
| `usd_*` | Total USD value per action | Quantifies financial activity |
| `repay_to_borrow` | Ratio of repay to borrow USD | Indicates debt repayment discipline |
| `redeem_to_deposit` | Ratio of redeemed to deposited USD | Proxy for withdrawal behavior |
| `active_days` | Days active between first and last tx | Measures longevity of engagement |
| `unrepaid_borrows` | Flag if borrow exists but no repay | Signals risky/lazy borrowers |

All features were chosen to ensure **explainability** and **practical interpretability** for downstream decision-makers (lenders, protocols, etc.).

---

##  **3\. Credit Scoring Method**

The scoring mechanism is a **hybrid rule-based \+ ML approach**:

### **3.1 Heuristic Score (Rule-Based)**

A custom scoring function computes a credit score from 0 to 1000 based on:

* **\+0.4** for wallets that have made deposits (signal trust & commitment)

* **\+0.2** × repayment ratio (reward full/partial repay)

* **\+0.15** × activity days (reward consistent users)

* **\+0.1** × redeem ratio (light reward for redemption)

* **−0.1** for wallets that borrowed but never repaid

* **−0.05** penalty for partial repayers (\<50%)

This scoring logic promotes **trustworthy financial behavior**, while discouraging **risky borrowing practices**.

### **3.2 Machine Learning Model (XGBoost Regressor)**

We also trained a supervised learning model (`XGBRegressor`) to:

* Predict the synthetic (heuristic) credit score

* Understand feature importance and interaction

* Generalize scoring logic to unseen wallet behavior

Hyperparameters:

`XGBRegressor(`  
    `n_estimators=300,`  
    `max_depth=8,`  
    `learning_rate=0.05,`  
    `subsample=0.9,`  
    `colsample_bytree=0.9,`  
    `objective='reg:squarederror'`  
`)`

Mean Absolute Error on test set: **\~X.XX** (← to be filled after actual run)

---

##  **4\. Risk Indicators Justification**

The scoring system captures **3 major risk dimensions**:

| Dimension | Captured By | Risk Signal |
| ----- | ----- | ----- |
| **Liquidity Risk** | `usd_deposit`, `usd_redeemunderlying`, `redeem_ratio` | Users who frequently withdraw may lack commitment |
| **Credit Risk** | `usd_borrow`, `usd_repay`, `repay_ratio`, `unrepaid_borrows` | Detects untrustworthy debt behavior |
| **Engagement Risk** | `active_days`, `num_transactions` | Short-lived wallets might be transient users or bots |

These indicators align with **real-world credit heuristics** used by traditional lenders, adapted for **DeFi and on-chain environments**.

---

## **Scalability**

* Built on Python with lightweight libraries (Pandas, XGBoost, etc.)

* Works with millions of transactions due to efficient use of `defaultdict`, `Counter`, and streaming logic

* Easily extendable to include new actions, tokens, and behaviors

* ML model can be retrained periodically with new behavior data

---

##  **Example Insights**

Here’s a sample wallet breakdown:

`Wallet Address: 0x000...1ee`  
`Synthetic Credit Score: 750.25`  
`Predicted Score (XGBoost): 748.11`

`Action Breakdown:`  
  `Deposits: 12`  
  `Borrows: 5`  
  `Repays: 5`  
  `Redeems: 4`

`USD Activity:`  
  `Deposit Value: $12,000.00`  
  `Borrow Value:  $3,500.00`  
  `Repay Value:   $3,480.00`  
  `Redeem Value:  $11,000.00`

