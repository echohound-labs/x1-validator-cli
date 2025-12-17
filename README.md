# EchoHound Labs — X1 Validator (CLI-Only)

A fully reproducible, CLI-only guide to running an X1 mainnet validator using Tachyon.  
Built from real mainnet deployment, real errors, and real fixes.

This repository documents a working, end-to-end X1 validator setup with no GUI dependency and full operator control. It is intended for validators already comfortable with Linux servers and CLI workflows.

All commands, configurations, and workflows contained here were executed, verified, and are actively running on X1 mainnet.

> ⚠️ Disclaimer  
> This repository documents validator setup and operation for educational and reproducibility purposes only and does not offer managed infrastructure, hosting services, delegation services, or operational support for third parties.

---

## Scope

This repository provides operator-grade infrastructure documentation for running an X1 validator via CLI.

It is intentionally:

- Minimal  
- Explicit  
- Auditable  
- Script-friendly  

---

## What this repository provides

- Ordered, end-to-end validator setup  
- CLI-only operation (SSH friendly)  
- Ledger-compatible authority handling  
- Proven X1 mainnet configuration  
- Troubleshooting based on real failure cases  

---

## What this repository does not provide

- Dashboards or GUIs  
- Staking strategy, ROI, or economics  
- Solana mainnet assumptions  
- Custodial tooling  

This is infrastructure documentation, not financial guidance.

---

## Repository structure

```

/README.md
/X1 Validator — Setup (CLI-Only).md
/X1 Validator — systemd Service Setup.md

```

Each document is intentionally self-contained and written to be followed in order.  
**Do not skip steps.**

---

## Environment assumptions

- Ubuntu / Debian-based Linux (Ubuntu recommended by X1)  
- Dedicated server (bare metal recommended)  
- Non-root operator user (e.g. `x1`)  
- SSH access  
- Comfort with terminal-only workflows  

---

## Validator proof-of-life (CLI)

A validator is considered running when the process is visible:

```bash
ps aux | grep tachyon-validator | grep -v grep
```

Expected output resembles:

```text
/home/x1/tachyon/target/release/tachyon-validator \
  --identity /home/x1/validator-keys/validator-keypair.json \
  --vote-account /home/x1/vote-account-keypair.json \
  --ledger /home/x1/validator-ledger \
  --snapshots /home/x1/validator-snapshots \
  --rpc-port 8899 \
  --dynamic-port-range 8000-8020 \
  --entrypoint <IP:PORT> \
  --expected-genesis-hash <HASH> \
  --log /home/x1/validator.log
```

If this process exists, the validator is live.

---

## Common warnings (normal behavior)

The following messages are expected and non-fatal:

* CUDA is disabled
* `net.core.rmem_max` too small
* I am not in the leader schedule yet

These do **not** indicate failure.

---

## Why EchoHound Labs published this

This documentation exists to:

* Reduce friction for new X1 validators
* Improve decentralization of the network
* Provide an operator reference that can be audited and trusted

Maintained by **EchoHound Labs**.
Community contributions welcome — clarity over cleverness.

```

