# permission-slip-openclaw-skill-google-calendar

An [OpenClaw](https://github.com/supersuit-tech) agent skill that lets an agent
check and manage the user's **Google Calendar** — entirely through the
[Permission Slip](https://github.com/supersuit-tech/permission-slip) CLI, so
every action respects the user's approval policy.

With this skill installed, a user can simply say **"check my calendar"** and the
agent lists today's events on their primary calendar via the Google connector,
using whichever Google account is set as the default in Permission Slip.

## How it works

This skill contains **no code of its own**. It is a thin instruction layer over
the `permission-slip` CLI:

- Authentication (Google OAuth), approval enforcement, and default-account
  selection all live in Permission Slip and its built-in `google` connector.
- The skill's only job is mapping natural language to the right CLI command
  (`permission-slip request --action google.<action> ...`) and presenting the
  JSON result.

Reads (`list_calendar_events`, `list_calendars`) are low-risk and typically
auto-approve. Writes (`create_calendar_event`, `update_calendar_event`,
`delete_calendar_event`, `create_meeting`) are approval-gated: the agent submits
the request and waits for the user to approve it in Permission Slip before
reporting success.

## Prerequisites

- An OpenClaw machine configured as an agent for a Permission Slip server.
- The `permission-slip` CLI installed and registered (`permission-slip whoami`
  should succeed).
- A connected Google account in Permission Slip with Calendar access. See the
  [Google connector docs](https://github.com/supersuit-tech/permission-slip/blob/main/connectors/google/README.md).

## Installation

Install the skill onto your OpenClaw machine when you're ready to use it — there
is no auto-install. Clone or copy `SKILL.md` into your agent's skills directory.

## See also

- [Permission Slip](https://github.com/supersuit-tech/permission-slip)
- [Google connector docs](https://github.com/supersuit-tech/permission-slip/blob/main/connectors/google/README.md)
- [permission-slip-openclaw-skill-protonmail](https://github.com/supersuit-tech/permission-slip-openclaw-skill-protonmail) — Proton Mail skill
- [permission-slip-openclaw-skill-gmail](https://github.com/supersuit-tech/permission-slip-openclaw-skill-gmail) — Gmail skill
- [permission-slip-openclaw-skill-google-drive](https://github.com/supersuit-tech/permission-slip-openclaw-skill-google-drive) — Google Drive skill
- [permission-slip-openclaw-skill-slack](https://github.com/supersuit-tech/permission-slip-openclaw-skill-slack) — Slack skill
