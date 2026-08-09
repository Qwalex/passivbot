---
name: passivbot-dev
description: >-
  Develop, review, and validate Passivbot changes safely. Use when editing trading
  logic, live orchestration, Rust strategy, exchanges, config, AI docs, backtests,
  or optimizer code; when choosing validation commands; or when unsure which
  docs/ai contract to read.
---

# Passivbot Development

## Session Start

1. Read `AGENTS.md`, `docs/ai/principles.md`, and `docs/ai/README.md`.
2. Route to only the contracts needed for the task (table in `docs/ai/README.md`).
3. Inspect branch, recent commits, worktree status, and relevant callers/tests before broad edits.

## Authority Gate

Stop and ask before:

- live / authenticated paper / testnet bot startup
- authenticated exchange calls (including read-only account probes)
- order create/cancel/modify
- credential use
- remote SSH / deploy / process control

Safe by default: local `fake` exchange, offline fixtures, Rust/Python unit tests, backtests on cached data.

## Ownership Boundary

| Layer | Owns |
|---|---|
| Rust (`passivbot-rust/`) | strategy, orders, risk, unstuck, backtest behavior |
| Python (`src/`) | orchestration, exchange I/O, config, data, reconciliation, gating |

Never fabricate required trading inputs. Never reimplement Rust intent in Python.

## Quick Router

| Task | Read |
|---|---|
| Failure / readiness / fallbacks | `docs/ai/error_contract.md` |
| Architecture / ownership | `docs/ai/architecture.md` |
| Validation / review | `docs/ai/validation.md` |
| Commands | `docs/ai/runbooks/commands.md` |
| Rust extension rebuild | `docs/ai/runbooks/rust_extension.md` |
| Exchanges | `docs/ai/features/exchange_integrations.md` |
| Strategy runtime | `docs/ai/features/strategy_runtime.md` |
| Candles | `docs/ai/features/candlestick_manager.md` |
| Fills / PnL | `docs/ai/features/fill_events_manager.md` |
| PR review loop | `docs/ai/runbooks/pr_review.md` |

## Validation Proportionality

Match validation to the changed contract (`docs/ai/validation.md`):

- docs → AI doc checks / claim verification
- Python orchestration → focused pytest + failure paths
- Rust behavior → Rust tests + extension rebuild/verify + parity
- exchange adapter → offline fixtures; no auth without approval
- config/schema → loader/roundtrip + example configs

Record commands run, outcomes, and intentionally untested surfaces.

## Public Output

Restate conclusions in public-repo-only terms. Keep private configs, hosts, credentials, and unreproducible local context out of commits and PR bodies.
