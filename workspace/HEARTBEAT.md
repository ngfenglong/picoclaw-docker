# Heartbeat Tasks

This file is read every 30 minutes. Execute the tasks below silently.

## Rules
- Do NOT send any message if everything is normal
- Do NOT send HEARTBEAT_OK as a message
- Only send a Telegram message if a threshold is breached or action is required

## Tasks

- Run `free -m` and check if used RAM exceeds 80% of total. If yes, send a Telegram message: "⚠️ RAM usage is above 80%: {used}MB / {total}MB"
- Run `df -h /` and check if disk usage exceeds 80%. If yes, send a Telegram message: "⚠️ Disk usage is above 80%: {used} / {total}"
