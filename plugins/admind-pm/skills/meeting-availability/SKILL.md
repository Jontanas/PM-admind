---
name: meeting-availability
description: Finds suitable meeting times by checking Outlook calendar availability for people named by the user or the participants of an email thread, and proposes a short list of free slots. When asked to book, creates the meeting in Scoro linked to the project, with activity type and billability, after the user confirms; Scoro syncs it to Outlook. Never books without an explicit request. Use when the user wants to schedule a meeting, find mutual availability, or propose time slots, e.g. "find a time for me, Anna and Piotr next week", "when are we all free for an hour?".
---

# Meeting Availability

This skill is **read-only** by default. Do not create a meeting unless the user explicitly asks you to schedule it.

Read calendars only through the Microsoft 365 connector — never by opening Outlook in a browser.

## Steps

1. **Identify the participants.**
   - Always include the user (the signed-in Microsoft 365 account) unless they say otherwise.
   - Add the people the user names. Resolve names to email addresses with the Microsoft 365 connector; if a name is ambiguous (two "Anna"s), ask.
   - If the request relates to an email thread, take the participants from that thread.
   - If it is unclear who should be included, ask the user.

2. **Ask before searching** (skip a question if the user already answered it):
   - Should the meeting be **30 minutes or 1 hour**?
   - Should I look for availability **this week or next week**?

3. **Check the Outlook calendars** of all participants for the requested period.

4. **Find overlapping free slots** that fit the requested duration.
   - Only propose slots within working hours (09:00–17:00 in the organiser's time zone, Monday–Friday) unless the user asks otherwise.
   - Do not propose slots that have already passed.
   - Avoid slots where any required participant is busy or out of office. A slot where someone is only *tentative* may be offered, but label it.

5. **Present a short list** of the best 3–5 options, each with:
   - Day and date
   - Start and end time
   - Time zone, when participants are in different zones
   - Whether each person is free or tentative in that slot

6. **Stop.** Only book the meeting if the user explicitly asks. If they do, follow "Booking the meeting" below.

## Booking the meeting — in Scoro

Meetings are created **in Scoro**, not in Outlook. Scoro then syncs the meeting to the participants' Outlook calendars. A Scoro meeting linked to the right project lets internal participants log their time on it, and it shows as a time slot in the Scoro Planner.

**Never open Outlook or Scoro in a browser** (web app, browser automation, or computer use). Use only the Scoro connector.

### 1. Work out the Scoro fields

- **Project** (required). Take it from the conversation, the email thread, or a project name or ID in the meeting title. Look it up in Scoro to get its ID and **project type** (Internal, One-off, Retainer). If it's unclear, search Scoro and offer up to 3 candidates. Never guess.
- **Activity type** (required). Suggest one from the project type, then let the user confirm or change it:
  - **Internal** project → **Internal project work**
  - **One-off** or **Retainer** project → **Client meetings - billable**

  Get the activity type IDs from Scoro's site configuration (activity types). If the suggested name doesn't exist, list the available activity types and ask.
- **Billable** — not asked; it follows the project type:
  - **Internal** → `non_billable`
  - **One-off** or **Retainer** → `billable`
  - Any other project type → ask the user.
- **Participants** — the **internal** participants (`@admindagency.com`) mapped to their Scoro users. If an internal person has no Scoro user, say so; they won't be in the Scoro meeting.
- **Title, start, end** — from the chosen slot, with the time zone.

### 2. Confirm everything once

Show one summary and wait for an explicit OK:

```
📅 [Title]
🕐 [Day, date] [start]–[end] [time zone]
📁 Project: [Project name] ([project type])
🏷️ Activity type: [activity type]
💰 Billable: [yes / no] (from the project type)
👥 Internal: [names]
🌐 External: [names/emails, or "none"]
```

> *"Reply ✅ to create this meeting in Scoro, or tell me what to change (project, activity type, time, people)."*

### 3. Create it in Scoro

On confirmation, create the calendar event with the Scoro connector:
- name, start and end (ISO 8601 with time zone)
- project ID, activity type ID, billable time type
- the internal participants' Scoro user IDs (including the user, if attending)
- a description with the agenda, if there is one

Report back with the event and its project. Don't create the event twice. If creation fails, show the error and don't retry with different fields without asking.

### 4. External attendees

A Scoro calendar event only takes Scoro users, so clients and other external people **can't be invited from Scoro**. If there are external attendees, tell the user after creating the event:

> *"Scoro will sync the meeting to Outlook for the internal team. Clients aren't Scoro users, so please add [emails] to the meeting in Outlook once it appears there."*

### If Scoro isn't available

If the Scoro connector isn't connected or can't create events, say so. As a last resort, offer a one-click Outlook link to a pre-filled meeting, and warn that it **won't be linked to the Scoro project**, so time can't be logged on it:

```
https://outlook.office.com/calendar/deeplink/compose?path=/calendar/action/compose&subject=<title>&startdt=<start>&enddt=<end>&to=<email1>,<email2>&body=<agenda>
```

URL-encode every value and use ISO 8601 date-times with the time-zone offset, e.g. `2026-10-01T10:00:00+02:00`.

## Notes

- If you can't read someone's calendar (external person, no permission), say so and give the options that work for everyone else.
- Keep the answer short: the list of options, plus one line for anything the user should know.
