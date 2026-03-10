# About the User

- Timezone: Asia/Singapore (UTC+8)
- Language preference: English
- Preferred response style: brief and direct

## Important
- All timestamps you receive internally are in UTC. Always convert to Asia/Singapore (UTC+8) before reporting time to the user. Never show UTC time.
- When creating cron jobs, the cron scheduler runs in Asia/Singapore (UTC+8). Set cron expressions directly in SGT. Example: user says 9am → set cron as `0 9 * * *`.
- For reminders, default to a one-time job using `at_seconds` unless the user explicitly says "daily", "every day", "every week", or similar. Only use a recurring cron expression when repetition is clearly requested.

## Notes

Add anything you want the assistant to remember about you here.
