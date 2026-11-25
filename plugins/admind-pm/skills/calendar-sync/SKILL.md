---
name: calendar-sync
description: Syncs the user's accepted, busy Outlook meetings (Mon-Fri, 08:00-18:00) into the Scoro calendar, linked to the right Scoro project, task and activity type, without creating duplicates. Always proposes the mapping first and writes to Scoro only after the user confirms in the Claude chat. Two ranges - WEEK (Mon-Fri of the current week) and DAY (today). Use for "sync my calendar to Scoro", "sync this week's meetings", "sync today's calendar", "link my calendar events to Scoro".
---

# Outlook → Scoro Calendar Sync

Run on demand. There are no scheduled runs.

The whole flow happens in the Claude chat: the proposal table, questions, confirmation and final report. Never send Slack messages or notifications to anyone.

**Golden rule: nothing is written to Scoro until the user has confirmed the proposal table (Step 6).**

## Pick the range

- **WEEK**: the user says "week", "this week" or "weekly", or it's Monday. Range: Monday to Friday of the current week, including days that have already passed (so past meetings get logged too). "Next week" moves the range forward one week.
- **DAY**: the user says "today" or "daily". Range: today only.
- **Specific dates**: if the user names a date or a range, use it, still Mon–Fri only.
- No range given: ask in one line: *"Today or the whole week?"*

---

## Step 1: Identify the user

Don't rely on names typed into the prompt.

- Microsoft 365 `get_me` gives the name, email and mailbox time zone.
- Scoro `get_me` gives the Scoro user ID.
- Internal domain: `@admindagency.com`.
- If the Microsoft 365 email and the Scoro email differ, stop and tell the user. Otherwise you might write into someone else's calendar.

Working time zone: the user's mailbox time zone (default **Europe/Warsaw**).

## Step 2: Fetch Outlook events

Fetch the user's events for the range. **Keep an event only if ALL of these are true:**

- The user **accepted** it, or is the **organizer**.
- showAs is exactly **busy**. Exclude free, tentative, oof and workingElsewhere.
- It isn't declined or cancelled.
- It falls on **Monday to Friday**.
- It overlaps **08:00–18:00** in the working time zone. An event that starts before 08:00 or ends after 18:00 but partly overlaps is kept. An event entirely outside those hours is skipped.
- It's **not an all-day event**.

For each kept event, note its **Outlook event ID**: `iCalUId` if available, otherwise `id`.

## Step 3: Fetch Scoro data

- The user's Scoro calendar events for the range, plus the **previous 4 weeks** (used as history in Step 5).
- Scoro users, to map internal emails to user IDs.
- Time off for all users in the range.
- Activity types from the site configuration, to check the IDs in the reference table.
- **The user's project pool**: active projects (not completed, invoiced, cancelled or deleted) where the user is project manager or member, or has tasks assigned. Also get the **open tasks** inside those projects, and each project's **client company** and **project type**.

### Reference: fixed Scoro projects and activity types

| Purpose | Scoro project (ID) | Activity type |
|---|---|---|
| Lunch | Admind operations (120) | from the matched task in the project |
| Scrum ceremonies | Admind Scrum (787) | from the matched task in the project |
| Internal meetings | Admind meetings (166) | from the matched task in the project |
| Trainings | Admind Trainings (2535), NOT "Admind Trainings_OLD" | from the matched task in the project |
| Any project, matched to a specific task | that project → task | the task's activity type |
| Billable project (One-off / Scoro "regular", or Retainer), no task | that project | Client meetings - billable (430) |
| Internal project, no task | that project | Internal project work (453) |

These four Admind projects (operations, Scrum, meetings, Trainings) are the **internal bucket projects**. If an ID no longer matches its name, look the item up by its exact name.

## Step 4: Match each Outlook event to an existing Scoro event

Every Scoro event this skill writes has this as the **last line of its description**:

```
[outlook-id: <Outlook event ID>]
```

Match in this order:

1. **By marker**: a Scoro event containing `[outlook-id: <this event's ID>]` on the same day.
2. **Fallback**: no marker match, but a Scoro event with the same title (ignoring case) on the same day **and no marker at all**. This catches older entries and copies made by Scoro's own Outlook sync. Adopt it: the marker is added when writing.

The result decides the action:

- **No match**: CREATE.
- **Match, different start/end or title**: UPDATE time and title.
- **Match, identical, already linked to a project**: SKIP. It's already in sync and isn't shown in the table except as a count.
- **Match with no project link**: LINK, using the mapping from Step 5.

**Scoro's own Outlook sync.** If the user says their Scoro Outlook (MS Exchange) sync is on, never CREATE. Only LINK or UPDATE the copies Scoro made. Events with no Scoro copy yet are listed as "waiting for Scoro's sync, run again later".

## Step 5: Propose project, task and activity type

Work out a proposal for every CREATE and LINK event. Apply the steps in order; the first one that gives an answer wins.

### 5a: Fixed rules

All keyword matching ignores case and matches whole words or phrases only.

1. **Personal fixed mappings.** These are standing rules the user has given, e.g. "Creative Tech Dailies → Admind Scrum, task Daily standup". Check the user's memory or project instructions, if available.
2. **Lunch**: title contains "lunch" or "lunching" → project **Admind operations**, task type **Break**, non-billable.
3. **Scrum**: the title contains *daily, standup, stand-up, scrum, sprint planning, sprint review, retro* or *retrospective* → project **Admind Scrum**, non-billable. Search the tasks by ceremony: daily or standup → the daily standup task; planning, review or retro → the matching task.
   - "Review" or "planning" on its own is **not** a scrum keyword. "ABB design review" goes to the project analysis in 5b.
4. **Internal meetings**: *1 on 1, 1:1, 1-on-1, GMA, Good Morning Admind* → project **Admind meetings**, non-billable.
5. **Trainings**: *training, webinar, knowledge sharing, lesson, lessons, wellbeing* → project **Admind Trainings**, non-billable.

**Internal bucket projects always get a task.** For rules 2–5, link the most relevant open task in that project, found by the meeting title (e.g. "Daily standup", "1:1", "GMA", the training's name). **Always use the activity type already set on that task.** Never type in an activity type by hand for these projects. If no task clearly fits, put the event under ❓ in Step 6 and list that project's open tasks to choose from.

**Admind meetings is used only through rule 4.** It is never a fallback for meetings that couldn't be matched.

### 5b: Project analysis (everything else)

Score each project in the user's project pool against the event. Use these signals, strongest first:

| Signal | Strength |
|---|---|
| Scoro project number or code in the title | Strong |
| The same recurring meeting (same title) was linked to a project in the last 4 weeks | Strong |
| Project name or a distinctive part of it in the title | Strong |
| Name of an open task in the project in the title or description | Strong (also gives the task) |
| External attendees' email domain belongs to the project's client company | Medium |
| Client company or brand name in the title | Medium |
| Project or task name in the description | Medium |

Generic words on their own ("Admind", "meeting", "sync", "call", "catch-up", "update", "weekly") are not evidence.

Confidence:

- **High**: one strong signal plus nothing pointing to another project, or two medium signals pointing to the same project.
- **Medium**: one medium signal, or two projects close together. Show both candidates.
- **Low or none**: no usable signal. Show up to 3 candidates from the pool that share the client domain, or "no candidate".

If a client company has several active projects and nothing in the title separates them, that's Medium. Show the candidates; don't pick one.

### 5c: Activity type and billable status

These rules apply to every event, whether it was matched in 5a, in 5b, or by the user in Step 6. The first rule that applies wins:

1. **Matched to a specific task** (in any project) → **always use the activity type already set on that task**.
2. **Internal bucket project** (Admind operations, Admind Scrum, Admind meetings, Admind Trainings) → always use the activity type of the relevant task in that project (see 5a). Never set it without a task.
3. **Billable project** (One-off / Scoro "regular", or Retainer), no specific task → **Client meetings - billable**.
4. **Internal project**, no specific task → **Internal project work**.
5. Any other project type, or a matched task with no activity type set → ask the user in Step 6.

Billable status follows the project type: **billable** for One-off and Retainer projects, **non-billable** for Internal projects and the internal bucket projects.

If the user changes the project or task in Step 6, work out the activity type again with these rules.

## Step 6: Confirm with the user (always, for WEEK and DAY)

Show **one table** with every CREATE, UPDATE and LINK event, numbered, grouped like this:

```
📅 Calendar sync — [week of 17–21 Nov / today, Mon 17 Nov]

✅ Ready to sync (fixed rules or high confidence)
 #  When              Event                     Action  Project → Task               Activity type            Billable  Why
 1  Mon 09:15–09:30   Creative Tech Dailies     create  Admind Scrum → Daily standup Admind Scrum             no        scrum keyword, type from task
 2  Mon 11:00–12:00   Hexagon | website review  create  Hexagon Website 2026         Client meetings-billable yes       project name in title

❓ Needs your choice
 3  Tue 14:00–15:00   Catch-up with Anna (CanPack)  create  a) CanPack Packaging Q4  b) CanPack Retainer 2026       client domain canpack.com
 4  Wed 10:00–10:30   Quick sync                     create  no candidate

🔄 Time changes only
 5  Thu 13:00 → 13:30 ABB weekly                     update  (project link unchanged)

⚠️ In Scoro but cancelled/declined in Outlook (not deleted, delete by hand if needed)
 –  Fri 10:00  Old kickoff

Skipped: 3 already in sync · 2 outside Mon–Fri 08–18 · 1 tentative
```

Then ask:

> *Reply **"ok"** to sync everything in ✅ and leave ❓ out, or give your changes, e.g. "3a, 4 → Admind operations, 2 → task Design review, 1 skip".*

Rules:

- Accept answers by number, letter or project name. Look up any project the user names in Scoro. If it's ambiguous, ask.
- **"skip"** → don't write that event.
- An event in ❓ that the user doesn't answer is **not written**. Remind them in the final report.
- If the user changes a lot, show the updated table once more before writing.
- If the user says something like "always map X to Y", apply it now. Tell them it'll be used as a personal fixed mapping next time (rule 5a-1).

## Step 7: Build the participant list

- User is the **organizer**: the user plus every attendee whose email ends in `@admindagency.com` and who has a Scoro user.
- User is only an **attendee**: the user only.
- Remove anyone with Scoro time off on the event's day.
- An internal email with no Scoro user is skipped; the event is still written.
- Never add external attendees.

## Step 8: Write to Scoro (confirmed events only)

**CREATE**

- Title: the Outlook subject.
- Start/end: same as Outlook, keeping the time zone.
- Description: the Outlook body (plain text, trimmed). The last line is the marker `[outlook-id: …]`.
- Participants: from Step 7.
- Project, related task, activity type, billable time type, and task type (Break for lunch only): as confirmed.

**UPDATE**: change start/end, and the title if it changed. Keep participants, the project link and the description. Add the marker if it's missing.

**LINK**: set project, task, activity type and billable status as confirmed. Add the marker if it's missing. Leave everything else unchanged.

Never change a project link that is already set; the user may have fixed it by hand. If the user explicitly asks to change it, do it.

If a write fails, carry on with the rest and list the failure in the report. Never retry by creating a second event.

## Step 9: Report

One short summary in the Claude chat:

```
✅ Synced: 6 created · 1 updated · 2 linked
⏸️ Not synced (no answer): #4 Quick sync
⚠️ Failed: none
⚠️ Cancelled in Outlook, still in Scoro: Old kickoff (Fri 10:00)
```

## Things this skill never does

- Writes to Scoro before the user confirms.
- Deletes Scoro events.
- Creates duplicates.
- Uses Admind meetings as a fallback.
- Adds external attendees as participants.
- Sends Slack messages or notifications to anyone.
- Opens Outlook or Scoro in a browser. It uses only the Microsoft 365 and Scoro connectors.
