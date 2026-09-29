# PM quickstart (about 5 minutes)

The skills are already installed for you by the Claude workspace admin. You only need to sign in to your tools and, optionally, turn on the calendar sync.

## 1. Connect your tools

In Claude, open **Settings → Connectors** and click **Connect** on each of these. Sign in with your Admind account.

- Microsoft 365 (Outlook mail and calendar, Teams)
- Scoro
- Google Drive
- Slack

Claude only sees what you can already see in each tool.

For `meeting-actions`, TeamsMaestro notes or recording must be on for your Teams calls.

## 2. Try the skills

Just ask in a chat. No setup needed:

- "Prep me for my call with Acme about the website project"
- "Find a time for me, Anna and Piotr next week"
- "Post a summary of my last Teams call to Scoro"
- "Process the latest email from Acme"
- "Turn this Scoro comment into a client email: …"
- "Run project intake on this brief" (attach the PDF/PPT)

If a skill doesn't kick in, name it: "use meeting-actions".

## 3. Turn on the calendar sync (optional)

Create two scheduled tasks (**Scheduled tasks → New task**), copying the prompt exactly from:

- [`scheduled-tasks/calendar-sync-weekly.md`](../scheduled-tasks/calendar-sync-weekly.md) — Mondays 09:00
- [`scheduled-tasks/calendar-sync-daily.md`](../scheduled-tasks/calendar-sync-daily.md) — weekdays 16:00

Turn on the Microsoft 365, Scoro and Slack connectors for both tasks, and turn on **automatic approval**. There is nothing to personalise, because the skill works out who you are. Use **Run now** once and check the result in Scoro.

When the sync can't tell which project an event belongs to, it sends you a Slack DM. Reply in Claude with, for example: "link my calendar events: 1 → Acme Website, 2 → no link".

**Tip:** put the Scoro project ID or name in meeting titles. It's the strongest signal for matching.

## Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| "I can't access Scoro / Outlook" | Connector not connected or signed out | Reconnect it in Settings → Connectors |
| Calendar sync creates nothing | Events are tentative or marked Free | Accept the invite and set it to Busy |
| Scheduled task didn't run | Laptop asleep with "requires your computer" on, or approvals set to ask | Turn off "requires your computer" or keep the app open; enable automatic approval |
| `meeting-actions` finds no notes | TeamsMaestro notes not ready yet | Wait a few minutes after the call, or paste the notes |
| A skill should work differently | — | Tell the team that maintains this repo. Don't fork your own copy, or you'll stop receiving updates. |
