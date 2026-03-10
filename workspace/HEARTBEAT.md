# Heartbeat Tasks

This file is read every 30 minutes. Execute the tasks below silently.

## Rules
- Do NOT send any message if everything is normal
- Do NOT send HEARTBEAT_OK as a message
- Only send a Telegram message if a threshold is breached or action is required

## Tasks

- Run `free -m` and extract the `available` and `total` values from the Mem row. Calculate usage as `(total - available) / total * 100`. If usage exceeds 80%, send a Telegram message: "⚠️ RAM usage is above 80%: {used}MB used, {available}MB available out of {total}MB"
- Run `df /` and read the `Use%` column for the `/` mount. If the percentage exceeds 80%, send a Telegram message: "⚠️ Disk usage is above 80%: {use%} used on /"
