# TxSignX Documentation

Architecture, security model, API and developer documentation for TxSignX.

## Status

Documentation repository setup only. Architecture, security model, and API
specifications are pending; Milestone 1 implementation has not started.

## Local workspace

```text
txsignx/                Local container; no Git repository
├── txsignx/            Independent Git repository: Rust engine
├── txsignx-web/        Independent Git repository: React application
├── txsignx-docs/       Independent Git repository: documentation
└── txsignx-brand/      Local only; no Git initialization, remote, commit, or push
```

The three GitHub repositories are:

- [j-kon/txsignx](https://github.com/j-kon/txsignx)
- [j-kon/txsignx-web](https://github.com/j-kon/txsignx-web)
- [j-kon/txsignx-docs](https://github.com/j-kon/txsignx-docs)

Each repository uses `main`. Never initialize Git in the outer workspace or in
`txsignx-brand/`. If Git metadata already exists in the brand directory, stop and
report it; do not delete it or alter history. Keep brand assets local.

## Setup verification

From the Rust repository, run `cargo fmt --all -- --check`,
`cargo check --workspace --locked`, `cargo test --workspace --locked`, and
`cargo clippy --workspace --all-targets --locked -- -D warnings`.

From the web repository, run `npm ci`, `npm run build`, and `npm run lint`.
This documentation repository contains Markdown and requires no build tooling.

## Repository hygiene

Never commit `.env` files, RPC credentials, seed phrases, mnemonics, private keys,
extended private keys, SQLite wallet databases, or API tokens. Inspect staged
files before committing. Ignore rules are a convenience, not a secret scanner.
Use normal pushes only; do not rewrite history.
