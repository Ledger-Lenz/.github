## Summary

<!-- What does this PR do? One or two sentences. -->

## Linked issue(s)

<!-- Closes #N -->

## Type of change

- [ ] Bug fix
- [ ] New feature
- [ ] Refactor / code quality
- [ ] Documentation
- [ ] CI / tooling
- [ ] Cross-repo change (affects shared contracts — see below)

## What was tested

<!-- Describe how you tested this. Include commands run and any notable edge cases. -->

```
# e.g.
pytest tests/test_benford.py -v
cargo test
```

## Cross-repo impact

<!-- Does this change affect any of the shared contracts below?
     If yes, list the linked PRs or issues in each affected repo. -->

- [ ] `RiskScore` struct (contract / core / data / api)
- [ ] Asset pair identifier format
- [ ] `submit_score` / `get_score` contract ABI
- [ ] Risk thresholds (`RISK_SCORE_FLAG_THRESHOLD`, `MAD_NONCONFORMITY_THRESHOLD`)
- [ ] Environment variable / config keys

Linked cross-repo PRs: <!-- e.g. Ledgerlens-contract#12, Ledegerlens-api#34 -->

## Checklist

- [ ] Tests pass locally (`pytest` / `cargo test`)
- [ ] Python: `black .` and `ruff check .` pass (Python repos)
- [ ] Rust: `cargo fmt --check` and `cargo clippy -- -D warnings` pass (contract repo)
- [ ] New code has tests
- [ ] Documentation updated where needed
- [ ] No secrets, `.env` files, or API keys committed
- [ ] PR title is concise and descriptive
