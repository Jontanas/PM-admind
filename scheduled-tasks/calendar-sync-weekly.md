# Scheduled task: Outlook → Scoro Calendar Sync (Weekly)

| Setting | Value |
|---|---|
| Name | Outlook → Scoro Calendar Sync (Weekly) |
| Schedule | Every Monday at 09:00 (Europe/Warsaw) |
| Connectors | Microsoft 365, Scoro, Slack |
| Automatic approval | On (otherwise the run stops and waits for a click nobody gives) |

## Prompt (copy exactly — nothing to personalise)

```
Use the calendar-sync skill in WEEKLY mode: sync my accepted, busy Outlook events for Monday to Friday of the current week into Scoro. This is an unattended run: do not ask questions in the chat. Send anything you can't resolve to me as a Slack DM, as the skill describes.
```

**If Scoro's own Outlook calendar sync (MS Exchange integration) is on for you**, add this sentence at the end of the prompt, so the task only links the meetings Scoro already copied instead of creating duplicates:

```
My Scoro Outlook sync is on.
```
