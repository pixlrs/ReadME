# Komi

Komi is a Discord moderation and utility bot controlled entirely with slash commands.

Operational handover context for future sessions is stored in `BOT_CONTEXT.md`.

## Features

- Utility: `/ping`, `/help`
- Moderation: `/ban`, `/unban`, `/kick`, `/timeout`, `/untimeout`
- Bulk delete: `/purge` (1-100 recent messages)
- Channel nuke: `/nuke` (one-shot clone + replace)
- Embeds:
  - `/embed` (simple embed builder)
  - `/embedjson` (paste full embed JSON)
- Access control:
  - `/access allow-user|allow-role|remove-user|remove-role|list`
  - Supports ID-based removal for deleted users/roles
- Ticket system:
  - Interactive manager panel for setup/configuration
  - Panel message can be published as buttons or dropdown
  - Multiple ticket types with optional emoji and optional dropdown description
  - 1 active ticket per user per ticket type
  - Ticket close confirmation (`Yes` / `No`)
- Channel flush:
  - Daily channel clone + replace jobs
  - `/flush add|remove|list`
- Logs:
  - `/log setup|panel|status`
  - Interactive setup panel (tabs: Logs / Roles / Channels)
  - Excluded channels + excluded roles support
- Reaction roles:
  - `/reactionrole add|remove|list`
  - Unicode emoji only
- Button roles:
  - `/buttonrole add|remove|list`
  - Click to toggle role
- Giveaways:
  - `/giveaway create|end|reroll|access`
  - Button-based entry with ephemeral join confirmation
- Sticky notes:
  - `/sticky panel` (primary UI with tabs, dropdown, and modals)
  - Keeps one bot note as the latest message in configured channels
  - Uses per-channel debounce delay from last matching message
- Runtime safety:
  - Single-instance runtime lock (prevents duplicate processing)
  - Ticket creation lock (prevents duplicate ticket channel creation)

## Requirements

- Node.js 18.17+
- A Discord bot token
- Discord bot permissions (depending on features you use):
  - Ban Members
  - Kick Members
  - Moderate Members
  - Manage Channels
  - Manage Roles
  - Send Messages
  - Embed Links
  - Read Message History
  - Add Reactions
  - Manage Messages (required for `/purge`)

## Setup

1. Install dependencies:

```bash
npm install
```

2. Create `.env`:

```env
DISCORD_TOKEN=your_bot_token
CLIENT_ID=your_application_id
# Optional (recommended for fast command updates in one server)
# GUILD_ID=your_test_server_id
# Optional: only enable if intent is enabled in Discord Dev Portal
# ENABLE_GUILD_MEMBERS_INTENT=true
# Optional branding/profile sync on startup
# BOT_NAME=Komi
# BOT_BIO=Komi can't communicate, but she wants to make 100 friends.
# Optional giveaway integrity signing override
# GIVEAWAY_AUDIT_SECRET=change_me_to_a_long_random_secret
# Optional temporary tamper test command gate
# ENABLE_GIVEAWAY_TAMPER_TEST=false
```

3. Deploy slash commands:

```bash
npm run deploy
```

- With `GUILD_ID`: deploys to one guild instantly
- Without `GUILD_ID`: global deployment (can take up to ~1 hour)

4. Start bot:

```bash
npm start
```

## Command Reference

### Moderation

- `/ping`
- `/help page:<number?>`
- `/ban user:<user> deletemessages:<0h|1h|6h|12h|24h> reason:<text?>`
- `/unban userid:<id> reason:<text?>`
- `/kick user:<user> reason:<text?>`
- `/timeout user:<user> minutes:<1-40320> reason:<text?>`
- `/untimeout user:<user> reason:<text?>`
- `/purge amount:<1-100> channel:<channel?>`
- `/nuke channel:<channel?> reason:<text?>`

Notes:
- `/unban` requires a Discord user ID, not mention or username.
- `/timeout` uses Discord preset durations: 60s, 5m, 10m, 1h, 1d, 7d.
- `/purge` can only bulk-delete messages newer than 14 days (Discord API limit).
- `/nuke` clones and deletes the original channel, then keeps the replacement in the same position.

### Embeds

- `/embed title:<text> description:<text> color:<hex|name?> channel:<channel?>`
- `/embedjson json:<payload> channel:<channel?>`

`/embedjson` accepts:
- Single embed object
- `{ "embeds": [...] }`
- Embed array `[...]`
- Optional fenced JSON code blocks

### Access

- `/access allow-user user:<user?> userid:<id?>`
- `/access remove-user user:<user?> userid:<id?>`
- `/access allow-role role:<role?> roleid:<id?>`
- `/access remove-role role:<role?> roleid:<id?>`
- `/access list`

Notes:
- Removal by ID works even if the user/role no longer exists.
- List output includes both mention and raw ID.

### Tickets

- `/ticket setup channel:<text-channel>`
- `/ticket panel channel:<text-channel?>`
- `/ticket open name:<prefix?> type:<ticket-type-key?>`
- `/ticket close`

Notes:
- `/ticket setup` stores the panel channel and opens the interactive ticket manager.
- `/ticket panel` re-opens the manager; optional `channel` updates panel channel first.
- There is no automatic default ticket type. New setups start with no types (`None`) until you add one.
- Ticket manager tabs:
  - `overview` (current config summary)
  - `settings` (panel type, support role, category, transcript channel)
  - `types` (add/remove types)
  - `actions` (publish panel message / refresh published panel message / toggle type-label visibility / set custom panel embed text)
- `Publish Panel Message` and `Refresh` require at least one configured ticket type.
- `Refresh` updates the last published ticket panel message; if no panel message exists yet, publish first.
- Panel embed type list (if enabled) shows ticket labels only, not internal type keys.
- Panel embed text can be customized from `Actions` -> `Set Panel Text`.
- Empty panel text input resets to the default:
  - `Choose a ticket type and the bot will create a private ticket channel.`
- Dropdown selection is reset after use.
- Ticket type keys are managed in the ticket manager `types` tab.
- Ticket channel naming format is:
  - `🎫・{ticket_type}-{username}`
- Ticket type `description` is shown in dropdown panels to explain the ticket purpose.
- Ticket type modal supports optional `Embed text` for the first message inside created tickets.
  - If left empty, the bot uses the default ticket-created embed text.
  - Supports placeholder `{username}` to mention/ping the ticket opener.
- Ticket type `emoji` must be a real Unicode emoji or custom emoji format (`<:name:id>` / `<a:name:id>`); `:alias:` text is ignored.
- Transcript channel is optional and configured in ticket manager `settings`.
- On close confirmation, the bot creates a transcript HTML, saves it locally in `transcripts/`, and stores transcript HTML + metadata in `ticket_transcripts`.
- If transcript channel is configured, the bot posts an embed summary + transcript HTML attachment there.
- Local transcript files older than 30 days are auto-deleted.
- Transcript HTML header icon now uses the current guild icon only (no template-logo fallback).
- Transcript summary embed follows `Ticket Closed` format (`Ticket Name`, `Opened By`, `Closed By`, `Date`, footer).

### Reactions and Buttons

- `/reactionrole add message:<message-link> emoji:<unicode-emoji> role:<role>`
- `/reactionrole remove message:<message-link> emoji:<unicode-emoji> role:<role>`
- `/reactionrole list message:<message-link?>`

- `/buttonrole add message:<message-link> role:<role> label:<text> style:<success|primary|secondary|danger?>`
- `/buttonrole remove message:<message-link> role:<role>`
- `/buttonrole list message:<message-link?>`

Notes:
- Reaction roles accept Unicode emojis only.
- Button roles toggle on click (add if missing, remove if already present).

### Giveaways

- `/giveaway create prize:<text> duration:<10m|2h|1d> winners:<1-20> channel:<channel?>`
- `/giveaway end message:<message-link>`
- `/giveaway reroll message:<message-link>`
- `/giveaway archive message:<message-link> note:<text?>` (archive a tampered giveaway after review)
- `/giveaway access action:<allow|remove|list> subject:<@user|@role?>`
- `/giveaway tamper message:<message-link> mode:<participants|winners|prize|all>` (temporary integrity test command; disabled by default)

Notes:
- Giveaway entry uses a button (no reaction roles).
- Users receive an ephemeral confirmation embed when they join.
- Duration format supports `s`, `m`, `h`, `d` (minimum `1m`, maximum `30d`).
- Winner draw uses cryptographic randomness (`crypto.randomInt`).
- End/reroll messages include an audit hash for the draw payload.
- Giveaway rows are additionally HMAC-signed when `GIVEAWAY_AUDIT_SECRET` is set; tampered rows are detected and blocked for end/reroll/join.
- If `GIVEAWAY_AUDIT_SECRET` is not set, the bot auto-creates and persists a fallback secret in `data/giveaway-integrity.secret` so integrity checks stay active.
- Missing giveaway signatures are treated as tampered (not auto-recreated) to prevent bypass by deleting `integrity_sig`.
- Giveaways are reloaded from SQLite before end/reroll/join checks, so manual DB edits are evaluated immediately.
- DB file is now force-refreshed from disk on giveaway reload paths, so external DB tools (for example DBPro) committed edits are actually visible to runtime checks.
- If integrity secret is unavailable at runtime, integrity checks fail closed (giveaways are treated as tampered/locked).
- On tamper detection, the bot publicly posts a tamper alert in the giveaway channel, locks the giveaway, and disables further resolution.
- Active giveaways are also checked by scheduler integrity scans (not only on giveaway-end timing), so tampering is auto-locked quickly.
- `/giveaway archive` moves a tampered giveaway into archived status (no resolution), updates the embed, and posts a public archive note.
- `/giveaway tamper` is for testing detection only and requires server admin plus `ENABLE_GIVEAWAY_TAMPER_TEST=true`.
- `/giveaway tamper` now immediately locks the target giveaway and posts the public tamper alert.
- Tamper mode `winners` now also mutates participants so forced-winner tampering tests are visible immediately.
- Giveaway delegated users/roles can create, end, and reroll giveaways.
- Only core bot-access users/roles can run `/giveaway access` to manage delegated-access lists.

### Sticky Notes

- `/sticky panel channel:<channel>`

Notes:
- `/sticky panel` is the primary control flow and includes:
  - tab navigation (content/behavior/actions)
  - mode dropdown
  - modal popups for text, embed JSON, and delay
- Sticky notes are re-posted after matching messages (based on mode) so the sticky remains near the bottom.
- The previous sticky message is deleted before posting the next one, so only one sticky message is kept per channel.
- Repost delay is per-channel and defaults to 60 seconds; new matching messages reset the timer.
- Trigger mode controls which message source resets the timer: all, users-only, webhooks-only, or bots-only.
- In `all` mode, bot messages are included (for example ticket panel/embed posts), while sticky self-posts are loop-suppressed.
- Sticky embed JSON must resolve to exactly one embed object (no extra `content` field).
- Sticky setup validates bot channel permissions before saving/posting and reports missing permissions.

### Channel Flush

- `/flush add time:<HH:MM> channel:<channel> timezone:<UTC offset hour?>` (e.g. `-1`, `0`, `+2`)
- `/flush remove channel:<channel>`
- `/flush list`

Notes:
- Flush jobs are stored by channel ID.
- If a channel is replaced (for example via `/nuke`) or recreated with the same name, flush jobs auto-reconcile to the replacement channel ID when possible.

### Logs

- `/log setup channel:<text-channel>`
- `/log panel`
- `/log status`

Logged categories:
- ban
- unban
- kick
- timeout
- role_create
- role_update
- role_delete
- role_add
- role_remove
- emoji_create
- emoji_update
- emoji_delete
- sticker_create
- sticker_update
- sticker_delete
- invite_create
- invite_use
- invite_delete
- channel_create
- channel_update
- channel_delete
- channel_nuke
- channel_flush
- voice_join
- voice_leave
- voice_move
- message_delete
- message_edit
- bulk_delete

Notes:
- The bot tries to include the executor ("who did it") via audit logs when available.
- Automated `/flush` operations emit dedicated `channel_flush` logs; related create/delete/position side-effect logs are suppressed.
- Bulk deletes are aggregated into a single log entry.
- Purge-triggered bulk deletes are suppressed from per-message spam.
- Message delete/edit logs use runtime message cache; message-content intent is enabled by default.
- Sticker create/update/delete logs include sticker emoji/tag; update logs include name and emoji/tag changes.
- Set `DISABLE_MESSAGE_CONTENT_INTENT=true` only if you explicitly want to disable it.
- In `/log setup`, exclusions are managed with grouped string dropdowns (roles/channels) so saved excluded items remain selected in the UI.
- `/log setup` includes a `Refresh` button to force a panel redraw for the current tab.

## Data Files

Persistent data is stored in `data/komi.sqlite` (SQLite database file).
For better compatibility with external DB editors (for example DBPro), runtime uses non-WAL journal mode.

The bot stores all operational datasets in this DB (access control, flush jobs, ticket settings, roles, logs, giveaways, sticky notes).
Data is organized in typed multi-table groups (not one blob row):
- Access: `access_control_typed`
- Flush: `flush_jobs_typed`
- Tickets: `ticket_guild_settings`
- Reaction roles: `reaction_role_rules_typed`
- Button roles: `button_role_rules_typed`
- Logs: `log_guild_settings_typed`
- Giveaways: `giveaways_typed` (includes participants/winners arrays on each giveaway row)
- Sticky notes: `sticky_notes_typed`
- Ticket transcripts: `ticket_transcripts` (stores HTML transcript content + metadata)

Legacy row/tag tables (and old compatibility tables) are migrated and pruned automatically; active reads/writes use the typed tables above.
If legacy `data/*.json` files exist from older versions, they are migrated into SQLite on startup and then removed.

Additional runtime artifacts:
- `runtime-logs/*.log` (one file per bot start, if run-log mirror enabled)
- `transcripts/*.html` (ticket transcripts, auto-pruned after 30 days)

## Scripts

- `npm start` - Run bot
- `npm run deploy` - Register slash commands
- `npm run check` - Syntax checks
- `npm run lint` - Alias of check
- `npm run test` - Local tests

Runtime logging notes:
- Console stdout/stderr is mirrored to `runtime-logs/run-YYYY-MM-DD_HH-mm-ss.log` at startup.
- A new timestamped log file is created for each bot start (no overwrite of previous runs).
- Failed starts that do not acquire the single-instance lock do not create runtime log files.
- Set `DISABLE_RUN_LOG_MIRROR=true` to disable runtime log mirroring.
