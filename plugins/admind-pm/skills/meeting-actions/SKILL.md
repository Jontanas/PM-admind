---
name: meeting-actions
description: Turns a Teams meeting transcript or TeamsMaestro AI notes into a structured Scoro comment (Key Highlights, Next Steps, Action Points with owners) and posts it to the right Scoro project after the user confirms. For external or mixed calls it also drafts a client follow-up email (never sends it). Works in the Claude app and in Slack. Use whenever someone asks to log, summarise, or post meeting notes to Scoro - "post the meeting notes to Scoro", "summarise today's call", "log my Teams call", "Teams transcript to Scoro", "get the summary of my meeting".
---

# Meeting → Scoro Summary

## When to use this skill

Invoke when the user wants to turn a meeting into a Scoro project comment. The meeting source is **TeamsMaestro** (Teams recordings processed via TeamsMaestro), the Teams transcript, or notes the user pastes.

The whole workflow — fetching, formatting, reviewing, posting — happens in the current conversation (a Claude chat or a Slack thread, wherever the user asked).

---

## Workflow

### Step 0 — Identify the meeting

Ask or infer from context which meeting to process:

- Named meeting → search by title or date.
- "Latest" / "last meeting" → fetch the most recent meeting that has already ended, and confirm: *"I found [Title] from [time] — is this the one?"*
- Multiple matches → list them: *"I found a few meetings — which one?"*
- Unclear → ask: *"Which meeting should I summarise — can you give me the title or date?"*

---

### Step 1 — Fetch the meeting content

Use the Microsoft 365 connector. Try these sources in order and stop as soon as you have content:

1. **Find the calendar event.** Search the Outlook calendar by meeting title, date, or participants, then read the full event. Note the attendee list and whether the event has a meeting transcript reference (e.g. a `meetingTranscriptUrl` field).
2. **Fetch the Teams transcript directly (preferred).** If the event has a transcript reference, read it using the connector's transcript resource, passing the value verbatim. Use speaker labels for action point ownership.
3. **Look for TeamsMaestro notes (fallback).** If no transcript is available, search the meeting chat (and, if needed, Outlook email) for messages from the "TeamsMaestro" bot with the AI-generated notes.
4. **Paste fallback.** If nothing is found, say: *"I can't reach the TeamsMaestro notes directly — please copy them from the TeamsMaestro app in Teams and paste them here."*

**If the transcript is not ready yet:** *"The TeamsMaestro notes usually appear a few minutes after the call ends — try again shortly."* Stop.

**Pasted / attached transcript:** accept it directly and go to Step 2.

---

### Step 2 — Identify the Scoro project

1. **User specifies a project** → look it up in Scoro (projects tool) to confirm ID and name.
2. **No project specified** → search Scoro with keywords from the meeting title and participants (a project name or ID in the meeting title is the strongest signal). Present up to 3 candidates: *"These projects could match — which one should I post to?"*
3. **No match found** → ask: *"I couldn't find a matching project — what's the Scoro project name?"*

Always confirm the project with the user before drafting anything.

---

### Step 3 — Determine meeting type

Check the attendee list and classify:

- **Internal** — only `@admindagency.com` attendees.
- **External / Mixed** — any attendee who is a client, prospect, or external stakeholder (even if internal people are also present).

If uncertain: *"Was this an internal meeting or a client call?"* This determines which drafts you prepare in Step 4.

---

### Step 4 — Prepare drafts

#### Always: Scoro comment draft

Focus on what is **new, decided, or actionable** — skip background and general discussion.

```
📋 Meeting Summary — [Meeting Title] | [Date]

✨ Key Highlights
• [2–4 bullets: major decisions, conclusions, key context]

🔜 Next Steps / New Developments
• [Concrete steps agreed, direction changes, newly scoped work]

✅ Action Points
• [Owner] — [Action] — [Due date if mentioned]
• [Owner] — [Action] — [Due date if mentioned]
```

Rules:

- Every Action Point needs a named owner. Use `[Owner TBC]` if unclear.
- Omit sections with no content — don't leave empty headings.
- No verbatim quotes — synthesise only.
- Plain language; keep technical terms from the meeting.

#### External / Mixed calls only: client follow-up email draft

When the meeting had any external attendees, **also prepare a follow-up email draft** alongside the Scoro comment — do not wait to be asked. Never include internal-only information (budgets, hours, internal disagreements, team names) in the client email, and never a Google Drive link (Admind's Drive is internal only; SharePoint links are fine).

```
Subject: [Meeting topic] — Follow-up

Hi [Client name / team],

Thank you for your time today. Here's a quick summary of what we covered and the agreed next steps.

[2–3 sentence overview from the client's perspective]

Next steps:
• [Action] — [Responsible party] — [Due date if applicable]

[Warm closing]

Best regards,
[User's name — from the signed-in account]
```

---

### Step 5 — Present drafts for review

**Internal meeting:**

> *"Here's the draft for **[Project Name]** in Scoro. Reply ✅ to post it, or tell me what to change."*

**External / mixed meeting:** show the Scoro comment first, then the email draft.

> *"Here are both drafts for **[Project Name]**. Reply ✅ to post the Scoro comment, or tell me what to adjust. The email draft is ready to copy — I won't send it on your behalf."*

Wait. Do not post to Scoro until the user confirms. Apply any edits and re-present until they are happy.

---

### Step 6 — Post to Scoro

On a clear affirmative reply ("yes", "post it", "looks good", "✅"), post the comment with Scoro's create-comment tool on the confirmed project. Reply:

*"Done — comment posted to **[Project Name]** in Scoro. ✓"*

---

## Error handling

- **Transcript not ready** — say so and ask them to try again shortly. Do not proceed.
- **Meeting not found** — ask them to check the title/date or paste the content directly.
- **No matching Scoro project** — ask for the project name explicitly.
- **Sparse transcript** — flag it: *"The summary is quite thin — no clear action points were captured. Want to add anything before I post?"*
- **Access/permissions error** — offer the paste fallback.

---

## What this skill does NOT do

- Does not post to Scoro without explicit user confirmation.
- Does not send emails — drafts only.
- Does not create Scoro tasks or time entries — project comments only.
- Does not run on a schedule — always user-initiated.
- Does not access video recordings — text transcripts and notes only.
