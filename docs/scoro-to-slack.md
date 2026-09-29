# Scoro → Slack notifications

The catalogue's third scheduled task, "Scoro → Slack notifications", was paused because of a connector issue. It is **not** included in the plugin. Even with the connector fixed, a scheduled Claude task is the wrong tool for this job:

- **One task per project, per PM.** It doesn't scale as projects start and end.
- **Deduplication by time window.** Each run starts with no memory and looks only at "the last 60 minutes". If a run is late, skipped (laptop asleep) or retried, comments are missed or posted twice.
- **It isn't real-time.** The shortest schedule is usually hourly.
- **It uses a Claude run every hour for every project,** even when nothing happened.

## Recommended: a small integration outside Claude

The job is "when a comment is added in Scoro, post a labelled message in the project's Slack channel". That is an event-driven integration:

```
Scoro (new comment) ──► integration ──► Slack channel for that project
```

**Trigger.** Use Scoro webhooks if your Scoro plan offers them for comments. Otherwise, poll the Scoro API every few minutes and **store the ID of the last comment processed** for each project. Deduplicating by stored ID, not by time window, is what makes it reliable.

**Project → channel mapping.** Keep it in one place, for example a Scoro custom field "Slack channel" on the project, or a small table in the integration. New projects then only need that field filled in. The `#project-…` channel created during project setup is the natural target.

**Labelling the comment type.** The six types from the catalogue (💬 Client feedback, ✅ Client approval, 📋 Meeting notes, 📁 New deliverable, 🔄 Project update, ✏️ Internal note) can be assigned:
- with simple keyword rules ("approved", "looks good", "go ahead" → Client approval), or
- with one small Claude API call per comment, if the rules aren't good enough.

**Message format.** Keep the catalogue's format:

```
[emoji] *[Comment type]* on *[Project Name]*
👤 Posted by: [Author full name]
🕐 [Time]
🔗 View in Scoro: [direct link]
```

**Where to build it.** A no-code tool (Make, Zapier, n8n) is enough if it has a Scoro connection. Otherwise use a small scheduled script (for example a serverless function) using the Scoro API and a Slack bot token. This needs someone technical for about a day. The original assessment reached the same conclusion.
