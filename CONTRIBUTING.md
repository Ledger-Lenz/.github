# Contributing to LedgerLens

Thank you for your interest in contributing to LedgerLens. This document applies to all repositories in the [Ledger-Lenz](https://github.com/Ledger-Lenz) organisation.

LedgerLens is an open-source fraud detection system for the Stellar DEX, built as a public good for the Stellar ecosystem. We welcome contributions of all kinds — bug reports, feature suggestions, documentation improvements, and code.

---

## Table of Contents

1. [Code of Conduct](#1-code-of-conduct)
2. [How to Contribute](#2-how-to-contribute)
3. [Development Setup](#3-development-setup)
4. [Coding Standards](#4-coding-standards)
5. [Testing](#5-testing)
6. [Pull Request Process](#6-pull-request-process)
7. [Cross-Repo Changes](#7-cross-repo-changes)
8. [Commit Style](#8-commit-style)

---

## 1. Code of Conduct

By participating in this project you agree to abide by the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). Be respectful, constructive, and collaborative.

---

## 2. How to Contribute

### Reporting bugs

Open an issue using the **Bug Report** template. Include:
- Which repository and version/commit is affected
- Steps to reproduce
- Expected vs actual behaviour
- Relevant logs or screenshots

### Suggesting features

Open an issue using the **Feature Request** template. Describe the problem you are trying to solve and why it matters for LedgerLens or the Stellar ecosystem.

### Asking questions

Open an issue using the **Question / Discussion** template, or join the conversation on [Stellar Discord](https://discord.gg/stellar).

### Contributing code

1. Open or find an issue describing the work
2. Comment on the issue to signal you are working on it
3. Fork the relevant repository and create a branch (`feat/short-description` or `fix/short-description`)
4. Make your changes following the standards below
5. Open a pull request against `main`

---

## 3. Development Setup

### Python repositories (`Ledgerlens-data`, `Ledgerlens-core`, `Ledegerlens-api`)

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Copy `.env.example` to `.env` and fill in any required values before running.

### Rust/Soroban repository (`Ledgerlens-contract`)

```bash
rustup target add wasm32-unknown-unknown
cargo build
cargo test
```

Requires Rust stable and the Soroban CLI. See [Soroban docs](https://soroban.stellar.org/docs) for setup.

### Dashboard (`Ledgerlens-dashboard`)

```bash
npm install
npm run dev
```

---

## 4. Coding Standards

### Python

- Formatter: **black** (`black .`)
- Linter: **ruff** (`ruff check .`)
- Type hints on all public functions
- Docstrings on all public classes and functions

### Rust / Soroban

- Formatter: **rustfmt** (`cargo fmt --check`)
- Linter: **clippy** (`cargo clippy -- -D warnings`)
- No `unwrap()` in production paths; use proper error propagation
- All public contract functions must have inline doc comments

### TypeScript / JavaScript (dashboard)

- Formatter: **prettier**
- Linter: **eslint**
- No `any` types without a comment explaining why

### General

- Keep commits focused — one logical change per commit
- Do not commit secrets, `.env` files, or API keys
- Update documentation alongside code changes

---

## 5. Testing

All pull requests must pass the full test suite for the target repository.

### Python repos

```bash
pytest
```

New features and bug fixes must include tests. Aim for meaningful coverage of the new code path, not just line coverage for its own sake.

### Rust/Soroban

```bash
cargo test
```

All contract functions must have at least one positive-path and one error-path test.

### Dashboard

```bash
npm test
```

---

## 6. Pull Request Process

1. **Target `main`** unless instructed otherwise
2. **Fill in the PR template** — summary, what was tested, linked issues
3. **Keep PRs focused** — one feature or fix per PR
4. **Pass CI** — all status checks must be green before review
5. **Request review** — tag a maintainer if there is no auto-assignment
6. **Respond to feedback** — address review comments or explain disagreement constructively
7. **Squash on merge** — the maintainer will squash and merge; keep the PR title clean

PRs that touch a [shared contract](#7-cross-repo-changes) must be coordinated across all affected repos.

---

## 7. Cross-Repo Changes

Some changes affect multiple repositories. Coordinate these carefully.

### Shared contracts

The following interfaces must stay in sync across all repos. Any change is a **breaking change** and requires linked PRs (or tracked follow-up issues) in every consuming repository:

| Contract | Defined in | Consumed by |
|---|---|---|
| `RiskScore` struct | `Ledgerlens-contract` (`types.rs`) | `Ledgerlens-core`, `Ledgerlens-data`, `Ledegerlens-api`, `Ledgerlens-dashboard` |
| Asset pair identifier format | `Ledgerlens-core` / `Ledgerlens-data` (`data_models.py`) | All repos |
| `submit_score` / `get_score` ABI | `Ledgerlens-contract` | `Ledegerlens-api`, any third-party Soroban integrations |
| `RISK_SCORE_FLAG_THRESHOLD` | `Ledgerlens-core` | `Ledegerlens-api`, `Ledgerlens-dashboard` |

### Process for cross-repo changes

1. Open an issue in `.github` describing the change and which repos are affected
2. Create a draft PR in each affected repo
3. Link all PRs to the `.github` issue
4. Merge in dependency order (contract → api → dashboard)
5. Close the tracking issue when all PRs are merged

---

## 8. Commit Style

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(benford): add MAD rolling window for 30d
fix(api): return 404 for unknown wallet instead of 500
docs(contract): clarify submit_score auth requirements
test(data): add round-trip feature extraction test
chore(deps): bump soroban-sdk to 25.3.1
```

Types: `feat`, `fix`, `docs`, `test`, `chore`, `refactor`, `perf`, `ci`

---

## Questions?

Open an issue with the **Question / Discussion** template or find us on [Stellar Discord](https://discord.gg/stellar).
