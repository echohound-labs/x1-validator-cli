### 👉 **`X1 Validator — systemd Service Setup.md` (FINAL, CLEAN)**

```md
# X1 Validator — systemd Service Setup (Tachyon)

This guide configures the X1 Tachyon validator to run as a persistent `systemd` service.
Using `systemd` ensures automatic restarts, clean shutdowns, and validator uptime across reboots.

---

## Goal

Run `tachyon-validator` as a managed system service that:
- starts automatically on boot
- restarts on failure
- shuts down cleanly
- survives SSH disconnects

---

## Assumptions

You already have:

- `tachyon-validator` built at  
  `/home/x1/tachyon/target/release/tachyon-validator`
- Validator identity and vote keys created and backed up
- Linux user: `x1`
- You are logged into the validator machine
- Validator directories already created:
  - `/home/x1/validator-ledger`
  - `/home/x1/validator-snapshots`

---

## 1 — Create the systemd service file

As **root** or via `sudo`:

```bash
sudo nano /etc/systemd/system/x1-validator.service
```

---

## 2 — Paste the service definition

⚠️ **Do NOT paste private keys.**
Use your real paths, but preserve this structure.

```ini
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
  --snapshots /home/x1/validator-snapshots \
  --rpc-port 8899 \
  --dynamic-port-range 8000-8020 \
  --entrypoint entrypoint.mainnet.x1.xyz:8001 \
  --expected-genesis-hash <GENESIS_HASH> \
  --wal-recovery-mode skip_any_corrupted_record \
  --log /home/x1/validator.log

ExecStop=/bin/kill -SIGINT $MAINPID
TimeoutStopSec=60

[Install]
WantedBy=multi-user.target
```

Replace `<GENESIS_HASH>` with the output of:

```bash
solana genesis-hash
```

---

## 3 — Reload systemd

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
```

---

## 4 — Enable the validator service

This ensures the validator starts on reboot.

```bash
sudo systemctl enable x1-validator
```

---

## 5 — Start the validator

```bash
sudo systemctl start x1-validator
```

---

## 6 — Confirm service status (CRITICAL)

```bash
systemctl status x1-validator
```

Expected:

```text
Active: active (running)
```

---

## 7 — Confirm process exists

```bash
ps aux | grep tachyon-validator | grep -v grep
```

You should see a live `tachyon-validator` process.

---

## 8 — Confirm network sync & voting

```bash
solana catchup <YOUR_VALIDATOR_PUBKEY> --url https://rpc.mainnet.x1.xyz
```

Expected:

```text
0 slot(s) behind
```

---

## 9 — Follow logs (optional)

Live systemd logs:

```bash
journalctl -u x1-validator -f
```

Or file-based logs:

```bash
tail -f /home/x1/validator.log
```

---

## Why systemd is the correct choice

Using `systemd` provides:

* Automatic restart on crashes
* Clean shutdown handling
* Persistent logging
* Survival across SSH disconnects
* Required stability for professional validator uptime

```




