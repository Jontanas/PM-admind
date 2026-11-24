# PM quickstart (about 5 minutes)

The skills are already installed for you by the Claude workspace admin. You only need to sign in to your tools.

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
- "Sync my calendar to Scoro" / "Sync this week's meetings"

If a skill doesn't kick in, name it: "use meeting-actions".

## 3. Calendar sync (on demand)

There are no scheduled calendar-sync tasks. When you want Outlook meetings linked in Scoro, ask in chat:

- "Sync today's calendar to Scoro"
- "Sync this week's meetings to Scoro"

Claude shows a proposal table (project / task / activity type). Nothing is written until you confirm. The whole flow stays in the Claude chat — no Slack DMs.

**Tip:** put the Scoro project ID or name in meeting titles. It's the strongest signal for matching.

## Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| "I can't access Scoro / Outlook" | Connector not connected or signed out | Reconnect it in Settings → Connectors |
| Calendar sync creates nothing | Events are tentative, Free, outside Mon–Fri 08:00–18:00, or all-day | Accept the invite, set Busy, and sync again |
| Calendar sync didn't write | You haven't confirmed the proposal table yet | Reply to confirm (or correct) the mapping in chat |
| `meeting-actions` finds no notes | TeamsMaestro notes not ready yet | Wait a few minutes after the call, or paste the notes |
| A skill should work differently | — | Tell the team that maintains this repo. Don't fork your own copy, or you'll stop receiving updates. |
