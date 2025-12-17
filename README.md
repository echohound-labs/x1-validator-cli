
EchoHound Labs — X1 Validator (CLI-Only)

A fully reproducible, CLI-only guide to running an X1 mainnet validator using Tachyon.
Built from a real mainnet deployment, real errors, and real fixes.

This repository documents a working, end-to-end X1 validator setup with no GUI dependency and full operator control. it is intended for validators comfortable with Linux servers and CLI workflows.

All commands, configurations, and workflows contained here were executed, verified, and are actively running on X1 mainnet.

⚠️ Disclaimer
This repository documents validator setup and operation for educational and
reproducibility purposes only and does not offer managed infrastructure, hosting services, delegation services, or operational support for third parties.

Scope

This repository provides operator-grade infrastructure documentation for running an X1 validator via CLI.

It is intentionally:

Minimal

Explicit

Auditable

Script-friendly

What this repository provides

✅ Ordered, end-to-end validator setup

✅ CLI-only operation (SSH friendly)

✅ Ledger-compatible authority handling

✅ Proven X1 mainnet configuration

✅ Troubleshooting based on real failure cases

What this repository does not provide

❌ Dashboards or GUIs

❌ Staking strategy, ROI, or economics

❌ Solana mainnet assumptions

❌ Custodial tooling

This is infrastructure documentation, not financial guidance.

Repository structure
/README.md
/01-prerequisites.md
/02-keypairs.md
/03-vote-account.md
/04-install-tachyon.md
/05-start-validator.md
/06-troubleshooting.md
/scripts/
  ├─ start-validator.sh
  └─ check-status.sh


Each file builds on the previous one.
Follow them in order. Do not skip steps.

Environment assumptions

Ubuntu / Debian-based Linux (Ubuntu recommended by X1)

Dedicated server (bare metal recommended)

Non-root operator user (e.g. x1)

SSH access

Comfort with terminal-only workflows

High-level setup flow

System preparation & tuning

Identity and authority keypairs

Vote account creation (Ledger supported)

Tachyon installation

Validator startup

Sync, health, and status verification

No step is optional.

Validator proof-of-life (CLI)

This repository intentionally avoids dashboards.
A validator is considered running when the process is visible.

ps aux | grep tachyon-validator | grep -v grep


Expected output resembles:

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


If this process exists, the validator is live.

Monitoring (CLI-only)

Follow logs in real time:

tail -f /home/x1/validator.log


Check vote account status:

solana vote-account /home/x1/vote-account-keypair.json

Common warnings (normal behavior)

The following messages are expected and non-fatal:

CUDA is disabled

net.core.rmem_max too small

I am not in the leader schedule yet

These do not indicate failure.

Official X1 resources & source references

This repository is built using official X1 documentation and source code, combined with real-world operator experience.

Official documentation

X1 Docs: https://docs.x1.xyz/

Hardware requirements:
https://docs.x1.xyz/validating/performance/hardware-requirements

Delegation portal:
https://delegation.mainnet.x1.xyz/

Official source code

X1 Labs GitHub: https://github.com/x1-labs

X1 Network GitHub: https://github.com/x1-network

Tachyon (validator client):
https://github.com/x1-labs/tachyon

⚠️ Note: The Anza Tachyon repository is not used for X1.
X1 maintains its own Tachyon fork.

Validator visibility

Community validator monitor: https://x1val.online/

Why EchoHound Labs published this

This documentation exists to:

Reduce friction for new X1 validators

Improve decentralization of the network

Provide an operator reference that can be audited and trusted

If this helps you, share it forward. Also, X1 tooling evolves — always cross-check with official docs.

Attribution

Maintained by EchoHound Labs

Community contributions welcome — clarity over cleverness.

👉 Next step:
01-prerequisites.md

