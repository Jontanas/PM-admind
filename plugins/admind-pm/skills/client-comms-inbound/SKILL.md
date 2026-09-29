---
name: client-comms-inbound
description: Handles inbound client communications from Outlook. Use when a client email needs to be processed for the internal team - identify the relevant Scoro project, summarise the client's message, save attachments to the project's Google Drive folder, draft a Scoro comment with action items and attachment links, and draft a warm, professional reply to the client. Always presents drafts for approval before posting or sending. Triggers - "process this client email for [project]", "log this email in Scoro", "handle the latest email from [client]".
---

# Client Communications — Inbound

Nothing is posted to Scoro and no email is sent until the user explicitly confirms. Saving attachments to Drive also waits for confirmation.

## Steps

1. **Find the email.** Use the email the user pasted or forwarded, or find it in Outlook with the Microsoft 365 connector (by sender, subject, or "latest from [client]"). If several emails match, list them and ask which one.

2. **Confirm the project.** Ask the user which Scoro project the email relates to. If you can suggest likely matches (a project name or ID in the subject, the client company), offer up to 3 candidates. Look the project up in Scoro to confirm its ID and name.

3. **Summarise the email** — what the client is asking, saying, or providing, in 2–5 bullets. Call out deadlines, approvals, and anything that changes scope.

4. **Find the Drive folder** for the project (a link in the Scoro project or task, or a search for the project name in Google Drive). Confirm the folder with the user. Attachments go into the `01_Client` subfolder when it exists (the standard Admind structure from `project-intake`); otherwise into the project root. Keep original file names.

5. **Draft the Scoro comment** for the internal team:

   ```
   📨 Client email — [Client name] | [Date]

   📋 Summary
   • [What the client is asking / saying / providing]

   ✅ Action items
   • [Owner or Owner TBC] — [Action] — [Due date if mentioned]

   📁 Attachments
   • [File name] → [Drive link]
   ```

   Use a friendly internal tone. Omit empty sections.

6. **Draft a reply email** to the client in a professional, warm tone. Acknowledge what they sent, confirm next steps and timing only where known, and never commit to dates, budget, or scope that the team has not agreed. Sign off with the user's name from the signed-in account.

7. **Present everything for approval:** the summary, the attachment plan, the Scoro comment, and the reply draft. Then ask:

   > *"Reply ✅ to save the attachments and post the Scoro comment, or tell me what to change. The email reply is a draft — I won't send it unless you ask me to."*

8. **On confirmation:** save the attachments, post the comment with the final Drive links, and report what was done with links. Only send or create an Outlook draft of the reply if the user explicitly asks.
