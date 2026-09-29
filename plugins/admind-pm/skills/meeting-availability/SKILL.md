---
name: meeting-availability
description: Finds suitable meeting times by checking Outlook calendar availability for people named by the user or the participants of an email thread, and proposes a short list of free slots. Read-only - never creates or sends invites unless explicitly asked. Use when the user wants to schedule a meeting, find mutual availability, or propose time slots, e.g. "find a time for me, Anna and Piotr next week", "when are we all free for an hour?".
---

# Meeting Availability

This skill is **read-only** by default. Do not create or send a calendar invitation unless the user explicitly asks you to schedule the meeting.

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

6. **Stop.** Only book the meeting if the user explicitly asks. If they do, confirm title, attendees and slot once before creating the invite.

## Notes

- If you can't read someone's calendar (external person, no permission), say so and give the options that work for everyone else.
- Keep the answer short: the list of options, plus one line for anything the user should know.
