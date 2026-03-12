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
  - Panel setup with buttons or dropdown
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

- `/ticket add-type key:<key> label:<label> description:<text?> emoji:<emoji?>`
- `/ticket remove-type key:<key>`
- `/ticket list-types`
- `/ticket setup channel:<text-channel> paneltype:<buttons|dropdown> supportrole:<role?> category:<category?>`
- `/ticket open name:<prefix?> type:<ticket-type-key?>`
- `/ticket close`

Notes:
- Dropdown selection is reset after use.
- Ticket type keys are shown in `/ticket list-types`.
- Ticket type `description` is shown in dropdown panels to explain the ticket purpose.

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
- `/giveaway access action:<allow|remove|list> subject:<@user|@role?>`

Notes:
- Giveaway entry uses a button (no reaction roles).
- Users receive an ephemeral confirmation embed when they join.
- Duration format supports `s`, `m`, `h`, `d` (minimum `1m`, maximum `30d`).
- Winner draw uses cryptographic randomness (`crypto.randomInt`).
- End/reroll messages include an audit hash for the draw payload.
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
- Set `DISABLE_MESSAGE_CONTENT_INTENT=true` only if you explicitly want to disable it.
