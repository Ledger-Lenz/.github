# LedgerLens 🔍

[![Built on Stellar](https://img.shields.io/badge/Built%20on-Stellar-blue?logo=stellar)](https://stellar.org)
[![Soroban Smart Contracts](https://img.shields.io/badge/Smart%20Contracts-Soroban-purple)](https://soroban.stellar.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://github.com/Ledger-Lenz/.github/blob/main/LICENSE)

### Hybrid On-Chain Fraud Detection for the Stellar DEX
**Detecting Wash Trading and Artificial Volume Using Benford's Law + Ensemble Machine Learning on Soroban**

---

> *"On a transparent ledger, every transaction is visible. LedgerLens makes them legible."*

---

## What We Build

LedgerLens is an open-source fraud detection system for the Stellar Decentralised Exchange (SDEX). It ingests trade data from the Stellar Horizon API, scores wallets and asset pairs for wash-trading risk, and exposes those scores through a public REST API, a web dashboard, and a Soroban smart contract so other protocols can query them natively.

Each wallet and trading pair receives a **LedgerLens Risk Score (0–100)** derived from:
- **Benford's Law digit-distribution analysis** — chi-square, per-digit Z-scores, and Mean Absolute Deviation across rolling time windows
- **Ensemble ML classifiers** (Random Forest, XGBoost, LightGBM) trained on 30+ on-chain features with SHAP interpretability

---

## Repositories

| Repository | Stack | What it does |
|---|---|---|
| [`.github`](https://github.com/Ledger-Lenz/.github) | YAML / Actions | Org-wide CI/CD, issue/PR templates, community health files |
| [`Ledgerlens-data`](https://github.com/Ledger-Lenz/Ledgerlens-data) | Python | Ingestion pipeline, Benford engine, ML features, ensemble training/inference |
| [`Ledgerlens-core`](https://github.com/Ledger-Lenz/Ledgerlens-core) | Python | Shared detection engine, local FastAPI for development |
| [`Ledegerlens-api`](https://github.com/Ledger-Lenz/Ledegerlens-api) | Python (FastAPI) | Public REST API — `/score`, `/alerts/recent`, `/assets/risk-ranking` |
| [`Ledgerlens-dashboard`](https://github.com/Ledger-Lenz/Ledgerlens-dashboard) | JS/TS (React) | Web dashboard — risk scores, alerts, asset risk rankings |
| [`Ledgerlens-contract`](https://github.com/Ledger-Lenz/Ledgerlens-contract) | Rust (Soroban) | On-chain risk score registry — `submit_score` / `get_score` |

---

## Why It Matters

Wash trading is cheap and easy on Stellar — 3–5 second finality, sub-cent fees, and native DEX infrastructure mean bad actors can inflate volume at scale. LedgerLens is the first open-source, production-grade detection layer for the SDEX.

- **Traders** get interpretable risk scores before placing orders
- **Asset issuers** can demonstrate organic volume
- **Protocol teams** can gate AMM and lending logic on LedgerLens scores natively via Soroban
- **The Stellar ecosystem** gains a credible, auditable fraud detection layer

LedgerLens is an **open-source public good** — scores, methodology, and training data are fully transparent and will always be free to query.

---

## Get Involved

We are actively looking for collaborators with experience in:

- Stellar / Soroban smart contract development (Rust)
- Python backend development and ML pipeline engineering
- On-chain data analysis and blockchain forensics
- Frontend development (React/TypeScript)
- DeFi protocol integration

Read [CONTRIBUTING.md](https://github.com/Ledger-Lenz/.github/blob/main/CONTRIBUTING.md) to get started, or open an issue in any repository.

---

*Built for the Stellar ecosystem as part of the Drip Wave builder programme. Open source. Community owned.*
