

### 👉 **`X1 Validator — Setup (CLI-Only).md` (FINAL, CLEAN)**

```md
# X1 Validator — Full CLI Setup (Start to Finish)

**Environment assumptions**
- OS: Ubuntu (recommended)
- Tested on: Ubuntu 24.04 LTS
- Network: X1 Mainnet

This document provides a complete, CLI-only walkthrough for deploying and running an X1 validator using Tachyon.  
Follow steps **in order**. Do not skip steps.

---

## 0 — Get a server

Review official X1 hardware requirements:

https://docs.x1.xyz/validating/performance/hardware-requirements

**Minimum expectations**
- High-core CPU
- Fast NVMe storage
- ≥ 128 GB RAM preferred
- Public IPv4

---

## 1 — Connect to your server (SSH)

From Windows PowerShell:

```bash
ssh root@YOUR_SERVER_IP
```

---

## 2 — Update system and reboot

```bash
apt update && apt upgrade -y
reboot
```

Reconnect after reboot.

---

## 3 — Create a non-root user (recommended)

```bash
adduser x1
usermod -aG sudo x1
su - x1
```

You should now see:

```text
x1@your-server:~$
```

---

## 4 — Install required system dependencies

These are **required** for Tachyon to build correctly.

```bash
sudo apt install -y \
  build-essential \
  pkg-config \
  libssl-dev \
  libudev-dev \
  clang \
  cmake \
  protobuf-compiler \
  curl \
  git
```

---

## 5 — Install Rust (required for Tachyon)

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env
```

Verify installation:

```bash
rustc --version
cargo --version
```

---

## 6 — Clone Tachyon (X1 validator client)

**IMPORTANT:** Use the X1 fork — **not** anza-xyz.

```bash
git clone https://github.com/x1-labs/tachyon.git
cd tachyon
```

---

## 7 — Build Tachyon (release mode)

This step takes time.

```bash
cargo build --release
```

If this completes successfully, continue.

---

## 8 — Verify Solana CLI (bundled with Tachyon)

From the Tachyon build directory:

```bash
./target/release/solana --version
```

Example output:

```text
solana-cli 2.2.18 (client:Tachyon)
```

---

## 9 — Configure Solana CLI for X1 mainnet

```bash
solana config set --url https://rpc.mainnet.x1.xyz
```

This setup assumes the Solana CLI is now pointed at X1 mainnet.

---

## 🔑 Key Management (CRITICAL)

### 10 — Create validator identity keypair (ON SERVER)

```bash
mkdir -p ~/validator-keys
solana-keygen new -o ~/validator-keys/validator-keypair.json
```

**Back up securely (offline):**

* 12-word seed phrase
* `validator-keypair.json`

---

### 11 — Create vote account keypair

```bash
solana-keygen new -o ~/vote-account-keypair.json
```

Back this up as well.

---

### 12 — (Optional but recommended) Withdraw authority via Ledger

Ledger is used **only** for withdraw authority — not validator identity.

Example (run on your PC):

```bash
solana-keygen pubkey "usb://ledger?key=16/0"
```

Example result:

```text
4oodRaiqr71VXvw2zAWR8GUJSsKu9WRCtVsjMvu8Vcsi
```

---

### 13 — Fund validator identity

Minimum recommended: **3 XNT**

```bash
solana address
```

Example:

```text
59KYnuMGduACCrCsydDZYm3HwPnrc1v5Qm1bVkF75uQe
```

Send XNT to this address.

---

### 14 — Create vote account (CORRECT COMMAND)

Run from your **PC** (Ledger signs withdraw authority):

```powershell
solana create-vote-account `
  "C:\path\to\vote-account-keypair-backup.json" `
  "C:\path\to\validator-keypair-backup.json" `
  usb://ledger?key=16/0
```

Verify:

```bash
solana vote-account ~/vote-account-keypair.json
```

Expected:

* Validator Identity ✔
* Vote Authority ✔
* Withdraw Authority = Ledger ✔

---

## 15 — Prepare validator directories

```bash
mkdir -p ~/validator-ledger
mkdir -p ~/validator-snapshots
```

---

## 16 — Choose entrypoints (known good)

Examples from live cluster gossip:

```text
185.189.46.20:8001
144.217.76.140:8000
51.79.78.13:8000
```

---

## 🚀 Start the validator

### 17 — Foreground (first test)

```bash
/home/x1/tachyon/target/release/tachyon-validator \
  --identity /home/x1/validator-keys/validator-keypair.json \
  --vote-account /home/x1/vote-account-keypair.json \
  --ledger /home/x1/validator-ledger \
  --snapshots /home/x1/validator-snapshots \
  --rpc-port 8899 \
  --dynamic-port-range 8000-8020 \
  --entrypoint 185.189.46.20:8001 \
  --entrypoint 144.217.76.140:8000 \
  --entrypoint 51.79.78.13:8000 \
  --expected-genesis-hash $(solana genesis-hash) \
  --log /home/x1/validator.log
```

---

### 18 — Background (production)

```bash
nohup /home/x1/tachyon/target/release/tachyon-validator \
  --identity /home/x1/validator-keys/validator-keypair.json \
  --vote-account /home/x1/vote-account-keypair.json \
  --ledger /home/x1/validator-ledger \
  --snapshots /home/x1/validator-snapshots \
  --rpc-port 8899 \
  --dynamic-port-range 8000-8020 \
  --entrypoint 185.189.46.20:8001 \
  --entrypoint 144.217.76.140:8000 \
  --entrypoint 51.79.78.13:8000 \
  --expected-genesis-hash $(solana genesis-hash) \
  --log /home/x1/validator.log \
  > /home/x1/validator.out 2>&1 &
```

---

## ✅ Verification (THIS IS HOW YOU KNOW IT WORKS)

**Process running**

```bash
ps aux | grep tachyon-validator | grep -v grep
```

**Logs moving**

```bash
tail -f ~/validator.log
```

**Vote account healthy**

```bash
solana vote-account ~/vote-account-keypair.json
```

---

## 🟡 Normal warnings (DO NOT PANIC)

Expected and non-fatal:

* CUDA is disabled
* `rmem_max` too small
* Not in leader schedule yet

These do **not** indicate failure.

---

## 🌐 Public monitoring

* Explorer: [https://x1val.online/](https://x1val.online/)
* Delegation: [https://delegation.mainnet.x1.xyz/](https://delegation.mainnet.x1.xyz/)

---

## Official sources used

* Docs: [https://docs.x1.xyz/](https://docs.x1.xyz/)
* GitHub: [https://github.com/x1-labs](https://github.com/x1-labs)
* Tachyon: [https://github.com/x1-labs/tachyon](https://github.com/x1-labs/tachyon)
* Network status: [https://x1val.online/](https://x1val.online/)

````

