---
name: calendar-sync
description: Syncs the user's accepted, busy Outlook calendar events into the Scoro calendar, linked to the right Scoro project and activity type, without creating duplicates. Three modes - WEEKLY (whole Mon-Fri week, used by the Monday scheduled task), DAILY (today only, used by the weekday 16:00 scheduled task), and RESOLVE (interactive - the user tells you which project unmatched events belong to). Use for "sync my calendar to Scoro", "run calendar-sync weekly/daily", "link my unresolved calendar events".
---

# Outlook → Scoro Calendar Sync

This one skill replaces the separate weekly and daily sync prompts from the catalogue. Both scheduled tasks call it, so a rule changed here applies to both.

## Pick the mode

- **WEEKLY** — the prompt says "weekly", or it is a Monday scheduled run. Date range: Monday to Friday of the current week.
- **DAILY** — the prompt says "daily" or "today". Date range: today only. This is a change scan: if nothing is new or changed, do nothing.
- **RESOLVE** — the user wants to link events that a previous sync couldn't match (see the last section).
- If the user runs it by hand without saying which, use DAILY and say so in one line.

## Unattended runs

WEEKLY and DAILY normally run as scheduled tasks with nobody watching. In those runs:

- Do not ask questions in the chat and do not wait for confirmation — nobody will answer.
- Anything you can't decide goes into the **unresolved list**, which is sent to the user as a Slack DM at the end (Step 9).
- Operate silently on success.

---

## Step 1 — Identify the user

Do not rely on names typed into the prompt. Determine:

- **Name, Scoro user ID, email** — from Scoro's "get current user" tool (`get_me`).
- **Outlook calendar** — the signed-in Microsoft 365 account.
- **Internal domain** — `@admindagency.com`.

If the Scoro user and the Microsoft 365 account don't have the same email, stop and report it (in a Slack DM on unattended runs) — you might otherwise write into the wrong person's calendar.

## Step 2 — Fetch Outlook events

Fetch the user's calendar events for the date range.

**STRICT FILTER — only include events where ALL of the following are true:**

- The user's response status is explicitly "accepted", **or** the user is the organizer
- The event's showAs / availability is exactly "busy"

**EXCLUDE** events where showAs is free, oof (out of office), tentative, or workingElsewhere. **EXCLUDE** events that are declined or cancelled.

**WEEKLY mode only:** skip events that have already started when the run happens (DAILY runs at 16:00 exist to catch those).

For each kept event, note its **Outlook event ID**: prefer `iCalUId` (stable across calendars and unique per occurrence of a recurring meeting); if not available, use the event `id`.

## Step 3 — Fetch existing Scoro data

- Scoro calendar events for the same date range.
- Scoro users (to map internal emails to user IDs).
- Time-off entries for all users for the date range.

## Step 4 — Match each Outlook event to Scoro

Every Scoro event this skill creates carries a marker as the **last line of its description**:

```
[outlook-id: <Outlook event ID>]
```

Match in this order:

1. **By marker** — a Scoro event whose description contains `[outlook-id: <this event's ID>]`.
2. **Legacy fallback** — no marker match, but a Scoro event with the same title (case-insensitive) on the same calendar day **and no marker at all**. This catches events created before the marker existed. Adopt it: add the marker to its description.

Then:

- **No match** → CREATE.
- **Match, but start/end time or title differs** → UPDATE time and title only. Never create a second entry.
- **Match, identical** → SKIP.

Because matching uses the ID, renaming an event in Outlook updates the Scoro event instead of creating a duplicate.

## Step 5 — Decide project, task and activity type

**"Lunch" events** — title contains "lunch" or "lunching" (case-insensitive):

- Task type: **Break**
- Activity type: **Admind operations**
- Project: the Scoro project named "Admind operations"

**All other events:**

1. Look for a Scoro project ID or project/task name in the event title, then in the description. A project ID in the title is the strongest signal.
2. If there is a high-confidence match, link the event to that project (and task, if one clearly matches) and pick the activity type:
   - matching task → the task's activity type
   - otherwise by project type: **Internal project work** for Internal projects; **Client meetings - billable** for One-off and Retainer projects
   - set billable by project type too: `non_billable` for Internal projects, `billable` for One-off and Retainer projects
3. If there is no confident match, **do not guess**. Create the event without a project link (so the time is still captured) and add it to the unresolved list. If Scoro refuses to create an event without a project, don't create it — just add it to the unresolved list.

On UPDATE, never change a project link that is already set (the user may have fixed it by hand).

## Step 6 — Build the participant list

- If the user is the **organizer**: the user + every attendee whose email ends in `@admindagency.com` and who has a matching Scoro user.
- If the user is only an **attendee**: the user only.
- Remove anyone with a Scoro time-off entry overlapping the event's day.
- An internal email with no Scoro user is skipped (the event is still created).
- Never add external attendees.

## Step 7 — Write to Scoro

**CREATE:**

- Title: the Outlook subject
- Start/end: same as Outlook, preserving the time zone
- Description: the Outlook body (plain text, trimmed), followed by the marker line `[outlook-id: …]` as the last line
- Participants: from Step 6
- Project/task, activity type, task type: from Step 5

**UPDATE:** change start/end (and title if it changed in Outlook). Keep participants, project link and description unchanged, except for adding the marker if it was missing.

## Step 8 — Things this skill never does

- Never deletes Scoro events. If an event with a marker no longer exists in Outlook (cancelled or declined), add it to the unresolved list as "cancelled in Outlook — delete in Scoro if needed".
- Never creates duplicates.
- Never touches other people's Scoro calendars beyond adding them as participants.

## Step 9 — Report unresolved events (Slack DM)

If the unresolved list is empty, finish silently.

Otherwise, find the user in Slack by their email and send one DM:

> Hi [first name]! I synced your Outlook calendar to Scoro ([this week / today]) but need your help with these events:
>
> 1. **[Event title]** — [Day, date, time] with [attendees] — *no matching Scoro project*
> 2. **[Event title]** — [Day, date, time] — *cancelled in Outlook, still in Scoro*
>
> To fix them, open Claude and say *"link my calendar events"*, e.g. "link my calendar events: 1 → [project], 2 → no link".

If Slack is unavailable, write the same list as the final output of the run.

---

## RESOLVE mode (interactive)

Used when the user answers the DM ("link my calendar events: 1 → Acme Website, 2 → no link"), or asks to fix unlinked events.

1. Find the user's Scoro events in the current and previous week that have an `[outlook-id: …]` marker but no project link.
2. Match the user's answers to those events (by number from the DM, or by title). If the user didn't give answers, list the unlinked events and ask.
3. Look up each named project in Scoro and pick the activity type with the rules in Step 5.
4. Show the planned changes and ask for confirmation, then update the events.
5. "No link" → leave the event as is.
