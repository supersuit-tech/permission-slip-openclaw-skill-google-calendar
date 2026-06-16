---
name: permission-slip-openclaw-skill-google-calendar
description: Check, look up, create, and manage events on a Google Calendar connected to Permission Slip. Use when the user says things like "check my calendar", "what's on my calendar today?", "am I free Thursday afternoon?", "schedule a meeting", or "move/cancel that event" and their Google account is managed via Permission Slip.
---

# Google Calendar (via Permission Slip)

This skill lets you act on the user's Google Calendar by driving the
`permission-slip` CLI. You never talk to the Google Calendar API directly —
every action goes through Permission Slip, which enforces the user's approval
policy.

This skill contains no code of its own: it is a thin shim over the
`permission-slip` CLI. All authentication (Google OAuth), approval enforcement,
and default-account selection live in Permission Slip and its built-in `google`
connector.

## Defaults

- **"Check my calendar" → list today's events on the primary calendar.**
  Run `list_calendar_events` with `calendar_id: "primary"` and `time_min` /
  `time_max` set to the start and end of *today* in the user's timezone
  (RFC3339, e.g. `2026-06-16T00:00:00-07:00` / `2026-06-16T23:59:59-07:00`).
- **"What's coming up / what's next" → upcoming events from now.**
  Set `time_min` to the current time and leave `time_max` unset.
- **Account selection:** omit `--instance`. Permission Slip auto-selects the
  user's *default* Google instance. Only pass `--instance` if the user
  explicitly names a second account.

## Preflight (run once per session, before the first action)

1. `permission-slip whoami` — confirm this agent is registered. If not, tell
   the user to register (`permission-slip register ...`) and stop.
2. `permission-slip connectors` — confirm `google` is available. If it's
   missing, the user hasn't connected a Google account yet; point them at the
   connector setup docs and stop.

## Intent -> action mapping

| User says | Action | Notes |
|-----------|--------|-------|
| "check my calendar", "what's on today?" | `google.list_calendar_events` | today's range, `primary` |
| "what's coming up / next?" | `google.list_calendar_events` | `time_min` = now |
| "am I free Thursday afternoon?" | `google.list_calendar_events` | range around the asked window |
| "which calendars do I have?" | `google.list_calendars` | read |
| "schedule / create an event" | `google.create_calendar_event` | **needs approval** |
| "set up a meeting (with a link)" | `google.create_meeting` | **needs approval** |
| "move / reschedule / edit that event" | `google.update_calendar_event` | **needs approval** |
| "cancel / delete that event" | `google.delete_calendar_event` | **needs approval** |

## `list_calendar_events` parameters

| Param | Default | Meaning |
|-------|---------|---------|
| `calendar_id` | `"primary"` | Which calendar to read |
| `max_results` | `10` (max `250`) | Cap on events returned |
| `time_min` | — | RFC3339 lower bound (inclusive) |
| `time_max` | — | RFC3339 upper bound; must be after `time_min` |

## How to run an action

```bash
permission-slip request \
  --action google.list_calendar_events \
  --params '{"calendar_id":"primary","time_min":"2026-06-16T00:00:00-07:00","time_max":"2026-06-16T23:59:59-07:00"}'
```

The CLI prints JSON. Two outcomes:

- **Auto-approved (reads):** the result includes the event list. Each event has
  an `id`, `summary`, `start`/`end`, `attendees`, and `htmlLink`. Use the `id`
  (with the same `calendar_id`) for any follow-up update/delete.
- **Pending approval (create / update / delete / meeting):** the CLI returns a
  request id in a `pending` state. Tell the user plainly: *"That needs your
  approval — I've sent the request to Permission Slip; I'll know once you approve
  it."* Then poll with `permission-slip request-status <id>` and report the
  outcome. Never claim an event was created or changed until the status is
  approved **and** executed.

## Presenting results

- Summarize the day briefly (time, title, attendees). Don't dump raw JSON unless
  asked.
- If the range returns no events, say the calendar is clear — don't retry blindly.
- Resolve relative dates ("today", "Thursday") against the user's timezone, and
  state the concrete date you used so they can correct you.
- On error, surface the connector's message verbatim and suggest the fix rather
  than guessing.

## Constraints

- `max_results` max is 250 — don't request more.
- One concern per request. For several distinct events, make one request each
  (or use bulk if the user wants a single approval prompt).
- Reads are low-risk; treat any write (create/update/delete/meeting) as
  approval-gated and communicate the wait.
