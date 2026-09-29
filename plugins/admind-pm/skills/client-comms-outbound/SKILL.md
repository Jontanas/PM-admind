---
name: client-comms-outbound
description: Handles outbound client communications based on internal Scoro updates. Takes an internal Scoro comment, task update, or project information, picks out what the client needs to know and any actions required from them, and drafts a clear, warm, professional client email for approval. Draft only - never sends. Use when an internal update needs to go to the client, e.g. "turn this Scoro comment into a client email", "draft a client update for [project]".
---

# Client Communications — Outbound

This skill **drafts only**. Never send the email. Only create an Outlook draft if the user explicitly asks.

## Steps

1. **Get the internal update.** Use what the user pasted, or read it from Scoro (the latest comments on the named project or task). If it is unclear which update to use, list the recent ones and ask.

2. **Identify the recipients** from the project's recent client email threads in Outlook, or ask the user.

3. **Separate client-relevant content from internal content.** Never include in the client email:
   - internal budgets, rates, costs, margins, or hours logged
   - internal team names, workload, or resourcing issues
   - blame, internal disagreements, or criticism of the client
   - internal-only links (Scoro, internal Drive folders, Slack)
   - anything the internal update marks as internal or confidential

   If the update contains something that looks client-relevant but sensitive (a delay, a scope change, extra cost), don't hide it and don't decide alone — flag it to the user and ask how to phrase it.

4. **Draft the email** with this structure (skip a part if it has no content):

   ```
   Subject: [Project name] — [Topic]

   Hi [Client name],

   [Update — what has happened / what's ready, in plain language]

   What we need from you:
   • [Action] — by [date]

   Next steps:
   • [What happens next] — [date]

   [Warm closing]

   Best regards,
   [User's name — from the signed-in account]
   ```

   Write in the language the client uses in their recent emails (default: English). Keep it short and warm.

5. **Present the draft** together with a one-line note of anything you deliberately left out as internal, so the user can check it. Apply edits until they are happy.
