# LedgerLens 🔍

[![CI — core](https://github.com/Ledger-Lenz/Ledgerlens-core/actions/workflows/ci.yml/badge.svg)](https://github.com/Ledger-Lenz/Ledgerlens-core/actions/workflows/ci.yml)
[![CI — data](https://github.com/Ledger-Lenz/Ledgerlens-data/actions/workflows/ci.yml/badge.svg)](https://github.com/Ledger-Lenz/Ledgerlens-data/actions/workflows/ci.yml)
[![CI — contract](https://github.com/Ledger-Lenz/Ledgerlens-contract/actions/workflows/ci.yml/badge.svg)](https://github.com/Ledger-Lenz/Ledgerlens-contract/actions/workflows/ci.yml)
[![Built on Stellar](https://img.shields.io/badge/Built%20on-Stellar-blue?logo=stellar)](https://stellar.org)
[![Soroban Smart Contracts](https://img.shields.io/badge/Smart%20Contracts-Soroban-purple)](https://soroban.stellar.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Hybrid on-chain fraud detection for the Stellar DEX — detecting wash trading and artificial volume using **Benford's Law** digit analysis and **ensemble machine learning**, with risk scores anchored on Soroban.

## Overview

LedgerLens is an open-source fraud detection system for the Stellar Decentralised Exchange (SDEX). It ingests trade data from the Stellar Horizon API, scores wallets and asset pairs for wash-trading risk, and exposes those scores through a public REST API, a web dashboard, and a Soroban smart contract so other protocols can query them natively.

Each wallet and trading pair receives a **LedgerLens Risk Score (0–100)** derived from Benford's Law digit-distribution analysis and ensemble ML classifiers (Random Forest, XGBoost, LightGBM). Scores update continuously as new ledger data is processed.

## The Problem

Wash trading — simultaneously buying and selling the same asset to artificially inflate volume — is one of the most pervasive forms of market manipulation in DeFi. Stellar's 3–5 second settlement finality and sub-cent fees make it cheap to execute at scale, while the volume of on-chain activity makes manual detection impossible.

On the Stellar DEX (SDEX), this causes real harm:

- **Traders are misled** into believing an asset has genuine liquidity when it does not
- **Token issuers manipulate volume rankings** on DEX aggregators by inflating 24-hour figures
- **Liquidity providers lose funds** by entering pools dominated by self-dealing activity
- **Ecosystem credibility suffers** — inflated metrics undermine confidence from institutional participants and new users

No production-grade, open-source wash-trading detection system existed for the SDEX. **LedgerLens fills that gap.**

## What LedgerLens Does

### 🔍 Detects
Identifies wallet pairs, trading clusters, and asset pools exhibiting statistically anomalous transaction patterns consistent with wash trading — circular trade routing, self-matching order behaviour, and artificial volume concentration.

### 📊 Scores
Assigns each wallet and each trading pair a **LedgerLens Risk Score (0–100)** based on the combined output of Benford anomaly metrics and ML classifiers. Scores update continuously as new ledger data is processed.

### 📡 Reports
Exposes risk scores and flagged activity through a public API and a lightweight dashboard, making intelligence accessible to DEX users, protocol teams, wallet providers, and compliance integrators without requiring any technical expertise.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     LAYER 1: DATA INGESTION                 │
│  Stellar Horizon API → trade history, order book events,    │
│  account activity, asset metadata                           │
│  Streamed continuously via SSE or polled per ledger close   │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  LAYER 2: DETECTION ENGINE                  │
│  ┌─────────────────────┐   ┌──────────────────────────┐    │
│  │  Benford's Law       │   │  Ensemble ML Models       │   │
│  │  Anomaly Engine      │   │  (RF, XGBoost, LightGBM) │   │
│  │  • Chi-square stat   │   │  • 30+ on-chain features  │   │
│  │  • Z-score per digit │   │  • SMOTE for imbalance    │   │
│  │  • MAD score         │   │  • SHAP interpretability  │   │
│  └──────────┬──────────┘   └──────────────┬────────────┘   │
│             └──────────┬──────────────────┘                 │
│                        ▼                                     │
│              LedgerLens Risk Score (0–100)                  │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               LAYER 3: SOROBAN CONTRACT + API               │
│  • Risk scores registered on-chain via Soroban contract     │
│  • Public REST API for external integrations                │
│  • Lightweight web dashboard for ecosystem visibility       │
│  • Webhook alerts for protocol teams                        │
└─────────────────────────────────────────────────────────────┘
```

## Repositories

This organisation is split across six repositories. Each can be developed independently; the shared contracts below keep them integrated.

| Repository | Stack | Responsibility |
|---|---|---|
| [`.github`](https://github.com/Ledger-Lenz/.github) | YAML / Actions | Org-wide CI/CD workflows, issue/PR templates, shared GitHub Actions |
| [`Ledgerlens-data`](https://github.com/Ledger-Lenz/Ledgerlens-data) | Python | Data ingestion pipeline, Benford's Law engine, ML feature extraction, ensemble training/inference, SHAP explanations, `RiskScore` computation |
| [`Ledgerlens-core`](https://github.com/Ledger-Lenz/Ledgerlens-core) | Python | Shared detection engine — Horizon ingestion, Benford analysis, ensemble training/inference; also hosts the local read-only FastAPI for development |
| [`Ledegerlens-api`](https://github.com/Ledger-Lenz/Ledegerlens-api) | Python (FastAPI) | Public REST API — serves `/score`, `/alerts/recent`, `/assets/risk-ranking`; the only repo with write access to the on-chain contract |
| [`Ledgerlens-dashboard`](https://github.com/Ledger-Lenz/Ledgerlens-dashboard) | JS/TS (React) | Web dashboard — visualises risk scores, alerts, and asset risk rankings by consuming the API |
| [`Ledgerlens-contract`](https://github.com/Ledger-Lenz/Ledgerlens-contract) | Rust (Soroban) | On-chain truth layer — `ledgerlens-score` Soroban contract storing the latest risk score per wallet/asset pair |

### End-to-End Data Flow

```
Stellar Horizon API
        │
        ▼
┌───────────────────┐
│  Ledgerlens-data   │  ingestion → detection → RiskScore records
│  Ledgerlens-core   │
└─────────┬──────────┘
          │ writes scored records (DB / queue)
          ├──────────────────────────────┐
          ▼                              ▼
┌───────────────────┐        ┌────────────────────────┐
│ Ledgerlens-       │ ◄───── │  Ledegerlens-api        │
│ contract          │  calls │  reads RiskScore store  │
│ submit_score()    │        │  reads on-chain scores  │
│ get_score()       │        └───────────┬────────────┘
└────────┬──────────┘                    │
         │ composable read               │ REST
         ▼                               ▼
  Other Soroban protocols       Ledgerlens-dashboard
  (AMMs, lending, aggregators)

.github — CI/CD workflows and community health files for all of the above
```

## Shared Contracts

These are the data shapes and conventions every repository must agree on. If you change one of these, open linked PRs in all consuming repos.

### `RiskScore` — canonical shape

```rust
pub struct RiskScore {
    pub score: u32,          // 0–100; higher = more suspicious
    pub benford_flag: bool,  // True if Benford anomaly detected
    pub ml_flag: bool,       // True if ML classifier flagged
    pub timestamp: u64,      // Ledger timestamp of last update
    pub confidence: u32,     // Model confidence 0–100
}
```

Defined in `Ledgerlens-contract` at `contracts/ledgerlens-score/src/types.rs`. Mirrored in `Ledgerlens-core`/`Ledgerlens-data` (`detection/risk_score.py`) and in `Ledegerlens-api` Pydantic schemas. **Any change to this struct is a breaking change across all six repos — coordinate via an issue here.**

### Asset pair identifier

Format: `CODE:ISSUER` (e.g. `USDC:GA5Z...`), or `XLM:native` for the native asset. Used consistently across API path parameters, the contract `Symbol` argument, and dashboard routing.

### Risk thresholds

| Constant | Default | Meaning |
|---|---|---|
| `RISK_SCORE_FLAG_THRESHOLD` | `70` | Score at or above this is surfaced as flagged in the API and dashboard |
| `MAD_NONCONFORMITY_THRESHOLD` | `0.015` | Benford MAD above this sets `benford_flag = true` |

### Contract interface

| Function | Caller | Auth | Used by |
|---|---|---|---|
| `initialize(admin, service)` | deployer | admin (one-time) | deployment tooling only |
| `submit_score(wallet, asset_pair, score, benford_flag, ml_flag, timestamp, confidence)` | LedgerLens service account | `service.require_auth()` | `Ledegerlens-api` — writes scores produced by the detection engine |
| `get_score(wallet, asset_pair)` | anyone | none (read-only) | `Ledegerlens-api`, `Ledgerlens-dashboard` (via api), and any third-party Soroban contract |
| `set_service(new_service)` | admin | `admin.require_auth()` | ops/admin tooling for key rotation |

`asset_pair` is a Soroban `Symbol` (≤ 9 chars, e.g. `XLM_USDC`). If pair identifiers need to exceed 9 characters, agree on a canonical short encoding before mainnet deployment.

## Benford's Law on the Blockchain

Benford's Law states that in naturally occurring numerical datasets the leading digit 1 appears ~30.1% of the time, declining to ~4.6% for digit 9. Genuine trading produces a wide, unbiased spread of transaction sizes that conforms to this distribution. Wash-trading bots — operating with fixed lot sizes, round numbers, or algorithmically generated amounts — deviate systematically from it.

| Metric | What it measures |
|---|---|
| **Chi-square statistic** | Whether the overall digit distribution deviates significantly from Benford's expected distribution |
| **Z-score (per digit)** | Whether any individual digit (1–9) appears with significantly higher or lower frequency than expected |
| **Mean Absolute Deviation (MAD)** | Composite divergence measure; values above 0.015 indicate non-conformity |

Benford signals alone are not definitive — legitimate high-frequency market makers can also produce non-Benford distributions — which is why LedgerLens combines them with the ML layer.

## Machine Learning Layer

### Feature groups (30+)

**Benford Features (15)** — chi-square, Z-score, and MAD for transaction amounts across 5 rolling windows (1h, 4h, 24h, 7d, 30d)

**Trade Pattern Features** — counterparty concentration ratio, round-trip trade frequency, self-matching rate, order cancellation rate and timing patterns

**Volume and Timing Features** — volume-to-unique-counterparty ratio, intra-minute clustering, off-hours activity ratio, volume spike frequency

**Wallet Graph Features** — funding source similarity, network centrality within trading clusters, account age at time of activity

### Models

| Model | Role |
|---|---|
| **Random Forest** | Stable baseline; handles missing features gracefully |
| **XGBoost** | Primary classifier; strongest performance on tabular on-chain data |
| **LightGBM** | High-speed inference for real-time scoring |

Models are trained with **SMOTE** to handle class imbalance and evaluated with **AUC-ROC**, **Precision-Recall AUC**, and **F1-score**. SHAP values provide per-score interpretability.

## Soroban Smart Contract Layer

The Soroban contract is the on-chain truth layer for LedgerLens risk scores. It provides two core functions:

- **`submit_score(...)`** — Called by the authorised LedgerLens off-chain service to register a computed risk score on-chain
- **`get_score(wallet, asset_pair) → RiskScore`** — Read-only, callable by any Soroban contract; returns the most recent risk score and timestamp

This composability means AMMs, lending protocols, and DEX aggregators on Stellar can integrate LedgerLens fraud signals natively — for example, gating liquidity provision from wallets with a risk score above a configurable threshold — without any off-chain dependency.

## API Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/health` | Health check |
| `GET` | `/score/{wallet}/{pair}` | LedgerLens Risk Score (0–100) for a wallet on an asset pair |
| `GET` | `/alerts/recent` | Wallet/asset-pair combinations currently flagged as high-risk, with reasons |
| `GET` | `/assets/risk-ranking` | Asset pairs ranked by aggregate wallet risk score |

## Roadmap

### Phase 1 — Foundation *(Months 1–2)*
- [x] Stellar Horizon API ingestion pipeline (historical + streaming)
- [x] Benford's Law engine for on-chain transaction amounts
- [x] Initial feature engineering from SDEX trade data
- [x] Baseline ML model training on synthetic wash trade patterns
- [ ] Internal testing on Stellar Testnet

### Phase 2 — Core Product *(Months 3–4)*
- [ ] Full ensemble model training and evaluation on labelled data
- [ ] SHAP interpretability integration
- [ ] Soroban smart contract deployment on Testnet
- [ ] Public REST API (v1) with rate limiting
- [ ] Web dashboard (beta)

### Phase 3 — Ecosystem Integration *(Months 5–6)*
- [ ] Mainnet deployment
- [ ] SDK for protocol integrations (Python + JavaScript)
- [ ] Webhook alert system for asset issuers and protocol teams
- [ ] Open dataset release: labelled SDEX wash trade patterns
- [ ] Community feedback and model refinement cycle

### Phase 4 — Scale *(Post-Grant)*
- [ ] Continuous model retraining pipeline
- [ ] Coverage expansion to AMM pools and cross-asset paths
- [ ] Integration partnerships with Stellar DEX aggregators
- [ ] Developer documentation portal

## Why This Matters for the Stellar Ecosystem

Stellar's growth as a platform for real-world asset tokenisation, remittances, and DeFi depends on the credibility of its markets. A DEX where volume figures cannot be trusted is one that institutional participants and serious traders will avoid.

**For traders** — Know which assets have genuine liquidity before placing orders. Risk scores provide instant, interpretable signals without requiring on-chain expertise.

**For asset issuers** — Demonstrate that your token's volume is organic. A low LedgerLens risk score is a credibility signal citable in listings and investor materials.

**For protocol teams** — Integrate LedgerLens scores into AMM and lending contract logic to automatically protect users from interacting with wash-traded assets or flagged wallets.

**For the Stellar Foundation and ecosystem** — An open, verifiable, community-maintained fraud detection layer strengthens the case for Stellar as trustworthy financial infrastructure.

LedgerLens is not a surveillance tool. It is an **open-source public good** — scores, methodology, and training data are fully transparent and auditable, and will always be free to query.

## This Repository

The `.github` repository provides organisation-wide defaults for all LedgerLens repositories:

- **`CONTRIBUTING.md`** — contribution guidelines applied to every LedgerLens repo
- **`SECURITY.md`** — vulnerability disclosure policy
- **`CODE_OF_CONDUCT.md`** — community standards
- **`.github/ISSUE_TEMPLATE/`** — bug report, feature request, and general issue templates
- **`.github/PULL_REQUEST_TEMPLATE.md`** — PR checklist
- **`.github/workflows/`** — reusable CI workflow templates for Python (lint/test) and Rust/Soroban (format/lint/test/wasm build)

## Contributing

LedgerLens is being developed as an open-source contribution to the Stellar ecosystem, submitted as part of the **Drip Wave builder programme**. We are actively looking for collaborators with experience in:

- Stellar / Soroban smart contract development (Rust)
- Python backend development and ML pipeline engineering
- On-chain data analysis and blockchain forensics
- Frontend development (dashboard)
- DeFi protocol integration

Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening an issue or pull request.

Quick checklist:
- Python repos: all tests pass (`pytest`) — formatting (`black .`) and linting (`ruff check .`) clean
- Rust/Soroban repos: all tests pass (`cargo test`) — formatting (`cargo fmt --check`) and linting (`cargo clippy`) clean
- New features include tests and documentation

## Security

To report a vulnerability, please follow the process described in [SECURITY.md](SECURITY.md). Do not open a public GitHub issue for security matters.

## References

- Benford, F. (1938) 'The law of anomalous numbers', *Proceedings of the American Philosophical Society*, 78(4), pp. 551–572.
- Al Ali, A. et al. (2023) 'A powerful predicting model for financial statement fraud based on optimized XGBoost ensemble learning technique', *Applied Sciences*, 13(4).
- Antonio, G.R. (2023) 'Numbers don't lie: Decoding financial error and fraud through Benford's law', *Journal of Entrepreneurship*.
- Nti, I.K. and Somanathan, A.R. (2024) 'A scalable RF-XGBoost framework for financial fraud mitigation', *IEEE Transactions on Computational Social Systems*, 11(2), pp. 410–422.
- Yadavalli, R. and Polisetti, R. (2025) 'Optimized financial fraud detection using SMOTE-enhanced ensemble learning with CatBoost and LightGBM', *ICVADV 2025*.
- Harea, R. and Mihailă, S. (2025) 'Benford's law: Applicability in accounting and financial anomaly detection', *Challenges of Accounting for Young Researchers*, 3(1).
- Stellar Development Foundation (2024) *Horizon API Documentation*. Available at: https://developers.stellar.org/api/horizon
- Stellar Development Foundation (2024) *Soroban Smart Contract Documentation*. Available at: https://soroban.stellar.org/docs

## License

MIT

---

<div align="center">

**LedgerLens** — Making the Stellar ledger legible.

*Built for the Stellar ecosystem. Open source. Community owned.*

</div>
