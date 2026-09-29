# Scheduled task: Outlook → Scoro Calendar Sync (Daily)

| Setting | Value |
|---|---|
| Name | Outlook → Scoro Calendar Sync (Daily) |
| Schedule | Weekdays at 16:00 (Europe/Warsaw) — move earlier if you finish before 16:00 |
| Connectors | Microsoft 365, Scoro, Slack |
| Automatic approval | On |

## Prompt (copy exactly — nothing to personalise)

```
Use the calendar-sync skill in DAILY mode: scan today's Outlook calendar and sync any new or changed accepted, busy events into Scoro. This is an unattended run: do not ask questions in the chat. If nothing changed, finish silently. Send anything you can't resolve to me as a Slack DM, as the skill describes.
```
