---
name: client-comms-outbound
description: Handles outbound client communications based on internal Scoro updates. Takes an internal Scoro comment, task update, or project information (including any Google Drive files linked in it, read through the Google Drive connector), picks out what the client needs to know and any actions required from them, and drafts a clear, warm, professional client email for approval. Draft only - never sends. Use when an internal update needs to go to the client, e.g. "turn this Scoro comment into a client email", "draft a client update for [project]".
---

# Client Communications — Outbound

This skill **drafts only**. Never send the email. Only create an Outlook draft if the user explicitly asks. Do that only through the Microsoft 365 connector; if it has no tool for drafts, give the user the text to copy. Never open Outlook in a browser.

## Steps

1. **Get the internal update.** Use what the user pasted, or read it from Scoro (the latest comments on the named project or task). If it is unclear which update to use, list the recent ones and ask.

2. **Read any linked Google Drive files.** If the update contains Google Drive links (`drive.google.com`, `docs.google.com`), read each file through the **Google Drive connector** — the user's own Drive account is connected, so you have access to what they can open.
   - **Never open or fetch the link as a web page** (web fetch, browser). Drive links need the user's sign-in, so that always fails. Always use the connector.
   - **Get the file ID from the link.** It is the long string after `/d/` or after `id=`:
     - `https://docs.google.com/document/d/<FILE_ID>/edit` (also `/spreadsheets/d/`, `/presentation/d/`)
     - `https://drive.google.com/file/d/<FILE_ID>/view`
     - `https://drive.google.com/open?id=<FILE_ID>`
     - A folder link (`/drive/folders/<FOLDER_ID>`) is not a file: list the files in that folder with the connector and read the relevant ones.
   - **Read the content** with the connector's "read file content" tool, passing the file ID. It handles Google Docs, Sheets and Slides, PDF, Word, Excel, PowerPoint and images. Use the metadata tool for the file name and type. Only if reading fails, use the connector's "download file content" tool.
   - If a file still can't be read (no access, unsupported type), say which one and why in one line, and continue with the rest. Don't invent what the file contains.
   - Use the file content to write a specific email: what was delivered, the key points or changes, and what the client needs to review or decide.

3. **Identify the recipients** from the project's recent client email threads in Outlook, or ask the user.

4. **Separate client-relevant content from internal content.** Never include in the client email:
   - internal budgets, rates, costs, margins, or hours logged
   - internal team names, workload, or resourcing issues
   - blame, internal disagreements, or criticism of the client
   - internal-only links (Scoro, internal Drive folders, Slack)
   - anything the internal update marks as internal or confidential

   If the update contains something that looks client-relevant but sensitive (a delay, a scope change, extra cost), don't hide it and don't decide alone — flag it to the user and ask how to phrase it.

5. **Draft the email** with this structure (skip a part if it has no content):

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

   **Files for the client — attach or link?** Claude can't attach files to emails: the Microsoft 365 connector doesn't support attachments. If the update includes files meant for the client, ask the user once, for all files together:

   > *"How should the client get [file names] — attached by you in Outlook, or as a Google Drive link in the email?"*

   Suggest **link** for videos and large files (over ~20 MB), which often exceed email size limits.

   - **Attach:** in the email body, add a marker where the file belongs, e.g. `[Attach: Homepage_v3.pdf]`, and refer to it as attached ("Please find the updated homepage design attached."). After the draft, list the files to attach, each with its Drive link so the user can download it quickly:

     ```
     📎 Attach before sending:
     • Homepage_v3.pdf → [Drive link]
     ```

     Remind the user to remove the `[Attach: …]` markers once the files are attached.
   - **Link:** before adding a Drive link, check the file's sharing settings with the Google Drive connector. If the client can't open it (shared only inside Admind), don't add the link silently. Tell the user and suggest they share the file with the client first, or switch to attaching it.

6. **Present the draft** together with a one-line note of anything you deliberately left out as internal, so the user can check it, and the "📎 Attach before sending" list if there is one. Apply edits until they are happy.
