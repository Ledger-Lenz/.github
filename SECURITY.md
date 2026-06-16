# Security Policy

## Supported Repositories

This policy applies to all repositories in the [Ledger-Lenz](https://github.com/Ledger-Lenz) organisation:

| Repository | Description |
|---|---|
| `Ledgerlens-contract` | Soroban smart contract (on-chain risk score registry) |
| `Ledgerlens-data` | Data ingestion and ML detection pipeline |
| `Ledgerlens-core` | Shared detection engine and local API |
| `Ledegerlens-api` | Public REST API |
| `Ledgerlens-dashboard` | Web dashboard |
| `.github` | Org-wide CI workflows and community health files |

## Reporting a Vulnerability

**Do not open a public GitHub issue for security vulnerabilities.**

If you discover a vulnerability, please report it privately so we can address it before it is disclosed publicly.

### How to report

Open a [GitHub Security Advisory](https://github.com/Ledger-Lenz/.github/security/advisories/new) in this repository. Your report will be visible only to maintainers.

Alternatively, email the maintainers directly. Contact details are available in the repository's `CODEOWNERS` file or on the org profile.

### What to include

A useful report includes:

- Which repository and component is affected
- A description of the vulnerability and the potential impact
- Steps to reproduce or a proof-of-concept
- Any suggested mitigations you have identified

### What to expect

| Timeframe | Action |
|---|---|
| Within 3 business days | Acknowledgement of your report |
| Within 10 business days | Initial assessment and severity triage |
| Within 30 business days | Patch or mitigation plan, or request for extension with explanation |

We will keep you informed throughout the process and credit you in the security advisory unless you prefer to remain anonymous.

## Scope

### In scope

- Logic errors in the `Ledgerlens-contract` Soroban contract that could allow unauthorised score submission, score manipulation, or fund loss
- Authentication or authorisation bypasses in any LedgerLens service
- Injection vulnerabilities (SQL, command, etc.) in the API or data pipeline
- Exposure of private keys, service secrets, or credentials through logs, error messages, or API responses
- Dependency vulnerabilities with a plausible exploit path against LedgerLens components

### Out of scope

- Theoretical vulnerabilities with no realistic exploit path
- Issues in third-party infrastructure (Stellar network, Horizon API, Soroban runtime) — please report those to the Stellar Development Foundation
- Denial-of-service attacks against the public API (rate limiting is a product concern, not a security issue)
- Issues already reported or known

## Smart Contract Specifics

The `Ledgerlens-contract` Soroban contract handles on-chain risk score registration. Key security properties:

- Only the authorised LedgerLens service account may call `submit_score`
- `get_score` is read-only and permissionless; it has no side effects
- Scores and confidence values are bounded to 0–100; the contract rejects out-of-range values
- Admin key rotation is restricted to the current admin via `set_service`

Reports of contract vulnerabilities that could allow manipulation of on-chain risk scores will be treated as **Critical** severity.

## Disclosure Policy

We follow a coordinated disclosure model:

1. Reporter submits a private report
2. We acknowledge, triage, and develop a fix
3. We release the fix and create a public GitHub Security Advisory
4. Reporter is credited (unless anonymity is requested)
5. A minimum of 7 days grace period after fix release before full public disclosure

We ask reporters not to publicly disclose the vulnerability until we have released a fix or 90 days have passed since the initial report, whichever comes first.

## Acknowledgements

We are grateful to everyone who takes the time to responsibly disclose vulnerabilities. Your efforts help keep the Stellar ecosystem safe.
