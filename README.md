
# Features

- Utility: `/ping`, `/help`
- Moderation: `/ban`, `/unban`, `/kick`, `/timeout`, `/untimeout`
- Bulk delete: `/purge` (1-100 recent messages)
- Embeds:
  - `/embed` (simple embed builder)
  - `/embedjson` (paste full embed JSON)
- Access control:
  - `/access allow-user|allow-role|remove-user|remove-role|list`
  - Supports ID-based removal for deleted users/roles
- Ticket system:
  - Panel setup with buttons or dropdown
  - Multiple ticket types with optional emoji
  - 1 active ticket per user per ticket type
  - Ticket close confirmation (`Yes` / `No`)
- Channel renewer:
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
- Runtime safety:
  - Single-instance runtime lock (prevents duplicate processing)
  - Ticket creation lock (prevents duplicate ticket channel creation)

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

Notes:
- `/unban` requires a Discord user ID, not mention or username.
- `/timeout` uses Discord preset durations: 60s, 5m, 10m, 1h, 1d, 7d.
- `/purge` can only bulk-delete messages newer than 14 days (Discord API limit).

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

- `/ticket add-type key:<key> label:<label> emoji:<emoji?>`
- `/ticket remove-type key:<key>`
- `/ticket list-types`
- `/ticket setup channel:<text-channel> paneltype:<buttons|dropdown> supportrole:<role?> category:<category?>`
- `/ticket open name:<prefix?> type:<ticket-type-key?>`
- `/ticket close`

Notes:
- Dropdown selection is reset after use.
- Ticket type keys are shown in `/ticket list-types`.

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

### Channel Flush

- `/flush add time:<HH:MM> channel:<channel> timezone:<UTC offset hour?>` (e.g. `-1`, `0`, `+2`)
- `/flush remove channel:<channel>`
- `/flush list`

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
- invite_create
- invite_use
- invite_delete
- channel_create
- channel_update
- channel_delete
- voice_join
- voice_leave
- voice_move
- message_delete
- message_edit
- bulk_delete

Notes:
- The bot tries to include the executor ("who did it") via audit logs when available.
- Bulk deletes are aggregated into a single log entry.
- Purge-triggered bulk deletes are suppressed from per-message spam.
- Message delete/edit logs use runtime message cache; message-content intent is enabled by default.
- Set `DISABLE_MESSAGE_CONTENT_INTENT=true` only if you explicitly want to disable it.