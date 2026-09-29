# Admind PM — Claude plugin for Client Service

This repository packages the skills from the *Admind AI Skills & Scheduled Tasks Catalogue* as a single Claude plugin (`admind-pm`). The Claude workspace admin installs it once for the organisation, so Project Managers don't copy and paste instruction blocks into their own accounts.

- **One source of truth.** Improve a skill here, push, and every PM gets the new version. With copy and paste, every PM kept their own copy.
- **Nothing to personalise.** Skills find the user's name, email and Scoro user ID from the connected accounts. There are no `[Your Full Name]` placeholders to replace.
- **Scheduled tasks are one-liners.** The task prompt just calls the `calendar-sync` skill. The weekly and daily syncs share the same rules and can't drift apart.

## What's inside

| Skill | Use it when | Say something like | Acts without asking? |
|---|---|---|---|
| `project-intake` | A new client brief or pitch request lands | "Run project intake on this brief" + attach files | No — waits for "go" |
| `pre-call-briefing` | About 30 min before a client call | "Prep me for my call with [client] about [project]" | Read-only |
| `meeting-availability` | You need to propose meeting slots | "Find a time for me, Anna and Piotr next week" | Read-only, never sends invites |
| `meeting-actions` | After a Teams meeting | "Post a summary of my last Teams call to Scoro" | No — waits for ✅ |
| `client-comms-inbound` | A client email needs processing | "Process this client email for [project]" | No — waits for ✅ |
| `client-comms-outbound` | An internal Scoro update must go to the client | "Turn this Scoro comment into a client email" | No — draft only |
| `calendar-sync` | Outlook → Scoro calendar sync (runs on a schedule) | "Sync my calendar to Scoro" / "Link my calendar events" | Yes, in scheduled runs — asks via Slack DM when unsure |

## Repository layout

```
.claude-plugin/marketplace.json        # marketplace the admin connects (lists the plugin)
plugins/admind-pm/
  .claude-plugin/plugin.json           # plugin manifest (bump "version" on every change)
  skills/<skill-name>/SKILL.md         # one folder per skill
scheduled-tasks/                       # copy-paste prompts for the two calendar-sync tasks
docs/
  admin-setup.md                       # one-time setup for the Claude workspace admin
  pm-quickstart.md                     # 5-minute setup for each PM
  scoro-to-slack.md                    # why Scoro → Slack notifications need an integration, and how
  changes-from-catalogue.md            # what changed compared to the original catalogue
```

## Getting started

- **Admin:** follow [`docs/admin-setup.md`](docs/admin-setup.md).
- **PMs:** follow [`docs/pm-quickstart.md`](docs/pm-quickstart.md).

## Changing a skill

1. Edit `plugins/admind-pm/skills/<skill>/SKILL.md`. The `description` in the frontmatter decides when Claude uses the skill. Put trigger phrases there.
2. Bump `version` in `plugins/admind-pm/.claude-plugin/plugin.json`.
3. Test the skill in a chat with a real case before merging to the default branch. With GitHub sync on, merging publishes it to everyone.

Keep the safety gates ("do not post until the user confirms", "draft only") unless the team has decided otherwise. They are part of Admind's governance, not a style choice.
