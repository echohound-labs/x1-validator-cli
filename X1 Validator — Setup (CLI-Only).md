X1 Validator — Full CLI Setup (Start to Finish)

Ubuntu is recommended and assumed
Tested on Ubuntu 24.04 LTS
Network: X1 Mainnet

0️⃣ Get a server

Follow X1 hardware requirements:

🔗 https://docs.x1.xyz/validating/performance/hardware-requirements

Minimum expectations:

High-core CPU

Fast NVMe storage

≥ 128 GB RAM preferred

Public IPv4

1️⃣ Connect to your server (SSH)

From Windows PowerShell:

ssh root@YOUR_SERVER_IP

2️⃣ Update system + reboot
apt update && apt upgrade -y
reboot


Reconnect after reboot.

3️⃣ Create a non-root user (recommended)
adduser x1
usermod -aG sudo x1
su - x1


You should now see:

x1@your-server:~$

4️⃣ Install required system dependencies

These are REQUIRED for Tachyon to build correctly.

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

5️⃣ Install Rust (for Tachyon)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env


Verify:

rustc --version
cargo --version

6️⃣ Clone Tachyon (X1 validator client)

IMPORTANT: Use the X1 fork — NOT anza-xyz.

git clone https://github.com/x1-labs/tachyon.git
cd tachyon

7️⃣ Build Tachyon (release mode)

This takes time.

cargo build --release


If this finishes successfully, you are good.

8️⃣ Install Solana CLI (bundled with Tachyon)

From the Tachyon build:

./target/release/solana --version


Example output:

solana-cli 2.2.18 (client:Tachyon)

9️⃣ Configure Solana CLI for X1 mainnet
solana config set --url https://rpc.mainnet.x1.xyz

Note: This setup assumes your Solana CLI is now configured to X1 mainnet.

🔑 Key Management (CRITICAL)
10️⃣ Create validator identity keypair (ON SERVER)
mkdir -p ~/validator-keys
solana-keygen new -o ~/validator-keys/validator-keypair.json


✔ Save:

The 12-word seed phrase

The validator-keypair.json

Back them up offline.

11️⃣ Create vote account keypair
solana-keygen new -o ~/vote-account-keypair.json


Back this up too.

12️⃣ (Optional but recommended) Withdraw authority via Ledger

Ledger is used ONLY for withdraw authority, not identity.

Example (done on your PC):

solana-keygen pubkey "usb://ledger?key=16/0"


Result:

4oodRaiqr71VXvw2zAWR8GUJSsKu9WRCtVsjMvu8Vcsi

13️⃣ Fund validator identity (minimum 3 XNT recommended)

Send XNT to:

solana address


Example:

59KYnuMGduACCrCsydDZYm3HwPnrc1v5Qm1bVkF75uQe

14️⃣ Create vote account (THIS IS THE CORRECT COMMAND)

Run from your PC (Ledger signs withdraw authority):

solana create-vote-account `
  "C:\path\to\vote-account-keypair-backup.json" `
  "C:\path\to\validator-keypair-backup.json" `
  usb://ledger?key=16/0


You should receive a transaction signature.

Verify:

solana vote-account ~/vote-account-keypair.json


Expected:

Validator Identity ✔

Vote Authority ✔

Withdraw Authority = Ledger ✔

15️⃣ Prepare validator directories
mkdir -p ~/validator-ledger
mkdir -p ~/validator-snapshots

16️⃣ Choose entrypoints (KNOWN GOOD)

Example working entrypoints:

185.189.46.20:8001
144.217.76.140:8000
51.79.78.13:8000


(Source: live cluster gossip)

🚀 FINAL — Start the validator
17️⃣ Foreground (first test)
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

18️⃣ Run in background (production)
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

✅ Verification (THIS IS HOW YOU KNOW IT WORKS)
Process running
ps aux | grep tachyon-validator | grep -v grep

Logs moving
tail -f ~/validator.log

Vote account healthy
solana vote-account ~/vote-account-keypair.json

🟡 Normal warnings (DO NOT PANIC)

These are expected:

CUDA is disabled

rmem_max too small

I am not in the leader schedule yet

They do NOT mean failure.

🌐 Public monitoring

Explorer: https://x1val.online/

Delegation: https://delegation.mainnet.x1.xyz/

Official sources used

Docs: https://docs.x1.xyz/

GitHub: https://github.com/x1-labs

Tachyon: https://github.com/x1-labs/tachyon

Network status: https://x1val.online/