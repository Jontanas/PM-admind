# Changes compared to the original catalogue

The skill instructions come from the *Admind AI Skills & Scheduled Tasks Catalogue*. The workflows, templates and safety gates are kept. These are the deliberate changes.

## All skills

- **No personal placeholders.** Skills identify the user from the connected accounts: Scoro "get current user" and the signed-in Microsoft 365 account. The internal domain `@admindagency.com` is fixed in the skills.
- **No hard-coded tool names.** The catalogue referenced names like `mcp__SCORO__get_projects`. The real prefix depends on how the connector is installed; in some setups it is `mcp__Scoro__…`, and a wrong name makes the step fail. Skills now describe the action ("look the project up in Scoro") and let Claude pick the tool.
- **Descriptions include trigger phrases**, so skills fire from normal wording.

## project-intake

- The Scoro task assignee defaults to the current user, instead of asking every time.
- The Drive folder is created with the `INTAKE` placeholder and **renamed with the real task code right after the Scoro task is created**. The original left the rename to the PM.
- **Choosing the Drive location (v2.0.1).** Before asking where to create the folder, the skill searches client shared drive → client folder → division folder, then offers up to 3 candidates (recommended first). Never defaults to Creative Technology when a client drive exists.

## pre-call-briefing

- The description is now a short "what and when" description. The original repeated the role text, which is less useful for triggering.
- With no client named, it finds the PM's next external meeting and confirms it.
- If a connector is unavailable, it continues with the others and says which source was missing.

## meeting-availability

- Always includes the user. Resolves ambiguous names by asking.
- Works within working hours (09:00–17:00, Mon–Fri) and never proposes past slots.
- Tentative participants can be offered, clearly labelled. The original excluded them entirely while also asking to show "tentative" in the output, which was contradictory.
- **Booking: Outlook first, then Scoro.** The invite is sent from Outlook through the Microsoft 365 connector, with a Teams link and clients invited (Scoro can do neither). Then the meeting is linked in Scoro to the project, so internal participants can log time on it and it shows in the Planner. Before sending, the PM confirms **project** and **activity type** (suggested from the project type). Billability follows the project type: Internal → non-billable, One-off/Retainer → billable. For PMs with Scoro's own Outlook sync, the synced copy is updated instead of creating a duplicate. If the connector can't create the invite, a pre-filled Outlook link is the fallback. Never books through a browser.

## meeting-actions

- **Claude chat only (v2.0.1).** The whole workflow stays in the current Claude chat; it no longer posts to Slack.
- "Latest meeting" means the latest meeting that has already ended.
- Follow-up emails never include internal-only information or Google Drive links.

## client-comms-inbound

- Adds a Scoro comment template, consistent with `meeting-actions`.
- Attachments go to `01_Client`, matching the `project-intake` folder structure.
- Saving attachments also waits for confirmation.
- The reply never commits to dates, budget or scope the team hasn't agreed.
- Outlook drafts are created only through the Microsoft 365 connector, never in a browser.

## client-comms-outbound

- Adds an explicit list of what must never reach the client: budgets, hours, internal team issues, blame, internal links.
- Sensitive but client-relevant items (delays, extra cost) are flagged to the PM, not silently dropped.
- Adds a standard structure: update → what we need from you → next steps.
- Writes in the client's language.
- **Finds Google Drive files linked in the Scoro update through the Google Drive connector** (file ID taken from the link), never by opening the link as a web page. The email's content still comes from the Scoro comment and the client thread; files are only read if the PM asks.
- **Never sends or suggests Google Drive links to clients** — Admind's Drive is internal only. Drive files for the client are attached by the PM: the draft marks where each goes and lists them with their Drive links for the PM only. SharePoint links can be sent. Files too large to attach are shared via SharePoint.

## calendar-sync (v2.0.1 — on demand, confirm first)

Replaces both the catalogue's weekly/daily sync prompts and the earlier plugin's unattended scheduled-task design.

- **On demand only.** No scheduled runs, no Slack DMs. Proposal, questions, confirmation and report all happen in the Claude chat.
- **Golden rule:** nothing is written to Scoro until the user confirms the proposal table.
- **Ranges:** WEEK (Mon–Fri of the current week, including past days), DAY (today), or a specific date range the user names.
- **Filters:** accepted/organizer, busy, Mon–Fri, overlapping 08:00–18:00, not all-day.
- **Richer matching:** project pool + open tasks, fixed internal bucket projects (operations, Scrum, meetings, Trainings), task activity types, and billability from project type.
- **No duplicates.** Each Scoro event stores `[outlook-id: …]`; title + day is the fallback for older entries and Scoro's own Outlook sync copies.
- **Never overwrites a project link set by hand, and never deletes events.**
- **Safety check:** stops if the Scoro user and the Microsoft 365 account have different emails.

## Scoro → Slack notifications

- Removed from the plugin. It is better built as a small integration; see [`scoro-to-slack.md`](scoro-to-slack.md).
