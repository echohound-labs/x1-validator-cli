X1 Validator — systemd Service Setup (Tachyon)

Goal:
Run tachyon-validator as a persistent, auto-restarting system service.

0️⃣ Assumptions

You already have:

tachyon-validator built at
/home/x1/tachyon/target/release/tachyon-validator

Validator keys stored securely

Linux user: x1

You are logged into the validator machine

1️⃣ Create the systemd Service File
sudo nano /etc/systemd/system/x1-validator.service

2️⃣ Paste the Service Definition

⚠️ DO NOT paste private keys  
Use your real paths, but keep this structure.

[Unit]
Description=X1 Tachyon Validator
After=network-online.target
Wants=network-online.target

[Service]
User=x1
Group=x1
LimitNOFILE=1000000
Restart=always
RestartSec=5
ExecStart=/home/x1/tachyon/target/release/tachyon-validator \
  --identity /home/x1/validator-keys/validator-keypair.json \
  --vote-account /home/x1/vote-account-keypair.json \
  --ledger /home/x1/validator-ledger \
  --log /home/x1/validator.log \
  --rpc-port 8899 \
  --dynamic-port-range 8000-8020 \
  --entrypoint entrypoint.mainnet.x1.xyz:8001 \
  --expected-genesis-hash <GENESIS_HASH> \
  --wal-recovery-mode skip_any_corrupted_record

ExecStop=/bin/kill -SIGINT $MAINPID
TimeoutStopSec=60

[Install]
WantedBy=multi-user.target

📌 Notes:

Restart=always → auto-restart on crash

LimitNOFILE → required for validators

ExecStop ensures graceful shutdown

3️⃣ Reload systemd
sudo systemctl daemon-reexec
sudo systemctl daemon-reload

4️⃣ Enable the Validator Service

This ensures it starts on reboot.

sudo systemctl enable x1-validator

5️⃣ Start the Validator
sudo systemctl start x1-validator

6️⃣ Confirm It’s Running (CRITICAL)
systemctl status x1-validator

You should see:

Active: active (running)

7️⃣ Confirm Process Exists
ps aux | grep tachyon-validator | grep -v grep

You should see a live tachyon-validator process.

8️⃣ Confirm Network Sync & Voting
solana catchup <YOUR_VALIDATOR_PUBKEY> --url https://rpc.mainnet.x1.xyz

Expected:

0 slot(s) behind

9️⃣ Optional: Follow Logs Live
journalctl -u x1-validator -f

Or log file:
tail -f /home/x1/validator.log

🧠 Why systemd Is the Correct Choice

Auto-restart on crashes

Clean logs

Safe shutdowns

Survives SSH disconnects

Required for professional uptime




