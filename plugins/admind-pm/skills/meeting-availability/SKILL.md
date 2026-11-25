---
name: meeting-availability
description: Finds suitable meeting times by checking Outlook calendar availability for people named by the user or the participants of an email thread, and proposes a short list of free slots. When asked to book, sends the invite from Outlook (with a Teams link, clients included) and links it in Scoro to the project, with activity type and billability, after the user confirms. Never books without an explicit request. Use when the user wants to schedule a meeting, find mutual availability, or propose time slots, e.g. "find a time for me, Anna and Piotr next week", "when are we all free for an hour?".
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

## Booking the meeting — Outlook first, then Scoro

The meeting is created **in Outlook first**, so it gets a **Teams link** and **clients can be invited** (Scoro can do neither). Then it is **linked in Scoro** to the project, so internal participants can log time on it and it shows as a time slot in the Scoro Planner.

**Never open Outlook or Scoro in a browser** (web app, browser automation, or computer use). Use only the Microsoft 365 and Scoro connectors.

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
- **Participants** — everyone goes on the Outlook invite. In Scoro, only the **internal** participants (`@admindagency.com`) mapped to their Scoro users. If an internal person has no Scoro user, say so.
- **Title, start, end** — from the chosen slot, with the time zone.

### 2. Confirm everything once

Show one summary and wait for an explicit OK:

```
📅 [Title]
🕐 [Day, date] [start]–[end] [time zone]
💻 Teams meeting: yes
📁 Project: [Project name] ([project type])
🏷️ Activity type: [activity type]
💰 Billable: [yes / no] (from the project type)
👥 Internal: [names]
🌐 External: [names/emails, or "none"]
```

> *"Reply ✅ to send the invite from Outlook and link it in Scoro, or tell me what to change."*

### 3. Create the meeting in Outlook

On confirmation, create the event in the user's Outlook calendar with the Microsoft 365 connector:
- title, start and end with the time zone, agenda in the body
- **all** participants as attendees, internal and external, so they receive the invite
- **a Teams online meeting**, if the create tool has that option

Then **read the event back** and check:
- **Teams link present?** If not, tell the user: *"The invite went out without a Teams link. Open the meeting in Outlook, switch on **Teams meeting**, and click **Send update**."*
- **All attendees on the invite?** If anyone is missing, say who.

Don't create the event twice. If creation fails, show the error and don't retry with different fields without asking.

**If the connector can't create events or can't add attendees**, don't create anything. Give the user a one-click Outlook link to a pre-filled meeting:

```
https://outlook.office.com/calendar/deeplink/compose?path=/calendar/action/compose&subject=<title>&startdt=<start>&enddt=<end>&to=<email1>,<email2>&body=<agenda>
```

URL-encode every value and use ISO 8601 date-times with the time-zone offset, e.g. `2026-10-01T10:00:00+02:00`. Tell the user: *"Check that **Teams meeting** is on, click **Send**, then tell me 'sent' so I can link it in Scoro."* When they say so, find the event in their Outlook calendar and continue with step 4.

### 4. Link it in Scoro — without creating a duplicate

Some PMs have Scoro's own Outlook calendar sync (MS Exchange integration) switched on; others don't. With it on, the Outlook meeting also appears in Scoro by itself, and a second event must not be created.

1. **Look for the synced copy.** Search the user's Scoro calendar for an event with the same title and the same start time.
   - **Found** → **update** that event: project, activity type, billable, internal participants, and add the marker line `[outlook-id: <Outlook event ID>]` at the end of the description. Done.
2. **Not found** → ask once: *"Is Scoro's Outlook calendar sync switched on for you? If you're not sure, say so."*
   - **Yes** → the copy may take a few minutes to appear. Say: *"I'll link it once it shows up in Scoro. Tell me 'link it' in a few minutes."* Then repeat step 4.1. If it still isn't there after a second check, offer to create it in Scoro instead.
   - **No** → **create** the event in Scoro with the Scoro connector: name, start and end, project ID, activity type ID, billable time type, the internal participants' Scoro user IDs, and a description ending with `[outlook-id: <Outlook event ID>]`.
   - **Not sure** → don't create anything. Suggest they check the MS Exchange calendar integration in their Scoro settings. Until then, running `calendar-sync` ("sync my calendar to Scoro") will pick the meeting up and link it.

Use the Outlook event's `iCalUId` as the Outlook event ID (or its `id` if `iCalUId` isn't available). The marker lets `calendar-sync` recognise the meeting later instead of copying it again.

### 5. Report back

One short confirmation: the Outlook invite (with or without a Teams link), who was invited, and the Scoro event with its project, activity type and billable status.

## Notes

- If you can't read someone's calendar (external person, no permission), say so and give the options that work for everyone else.
- Keep the answer short: the list of options, plus one line for anything the user should know.
