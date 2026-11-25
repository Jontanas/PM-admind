# Admin setup (one time)

For the Claude workspace admin (Owner / Primary Owner). These steps need a **Claude Team or Enterprise** plan with all PMs in the same organisation. With individual Pro accounts there is no central admin, and each PM would have to install everything by hand.

Menu names change from time to time. If a step doesn't match what you see, check the current Claude help centre article on managing plugins and connectors for your organisation.

## 1. Connectors (so PMs only click "Connect")

Enable these for the organisation:

| Connector | How | Notes |
|---|---|---|
| Microsoft 365 | From the connector directory | A Microsoft Entra (Azure AD) admin must grant tenant consent once. After that, each PM signs in with their own Admind account. |
| Scoro | Custom connector, using the Scoro MCP server URL | Each PM signs in with their own Scoro account, so Claude only sees what that PM can see in Scoro. |
| Google Drive | From the connector directory | |
| Slack | From the connector directory | Used by `pre-call-briefing` (and optionally other skills). Calendar sync no longer sends Slack DMs. |

**Microsoft 365 write actions.** `meeting-availability` sends meeting invites from Outlook (with a Teams link), and the client-comms skills create email drafts, so allow calendar and draft write actions in **Organization settings → Connectors → Microsoft 365 → Tool permission restrictions**. Your Entra admin must grant consent for the added permissions once (Enterprise applications → "M365 MCP Server for Claude" → Permissions → Grant admin consent), and each PM reconnects Microsoft 365. Without write actions, `meeting-availability` falls back to a pre-filled Outlook link.

Never collect PMs' passwords or API tokens centrally. Each person authorises their own access through OAuth.

## 2. Plugin (so PMs get the skills without copy and paste)

In **Organization settings → Plugins & skills**:

1. Add a plugin source and choose **sync from GitHub**, pointing at this repository (`.claude-plugin/marketplace.json` at the root). The org stays in sync with the default branch. If GitHub sync isn't available to you, zip `plugins/admind-pm/` and use **Upload a plugin** instead. You then need to re-upload after each change.
2. Set `admind-pm` to **Installed by default** (PMs can turn it off) or **Required** (always on).
3. On Enterprise you can limit it to a Client Service user group.

## 3. Governance decisions to make once

- **Confirm-before-write.** Skills that change Scoro (calendar sync, meeting notes, intake, inbound) propose first and wait for the PM in chat. Keep that gate unless the team decides otherwise.
- **Who can change the skills.** Protect the default branch and require a review. A merged change reaches every PM.
- **Security, privacy and legal review** of the connectors. The original assessment listed these as out of scope.

## 4. Rollout

1. Pilot with 2–3 PMs for a week. They follow [`pm-quickstart.md`](pm-quickstart.md).
2. Collect feedback and adjust the skills in this repo.
3. Enable for all Client Service.
