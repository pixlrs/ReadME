# DiscordBotMedia

DiscordBotMedia is a Discord moderation and utility bot controlled entirely with slash commands.

## Features

- Moderation: `/ban`, `/unban`, `/kick`, `/timeout`
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
  - `/renew add|remove|list`
- Reaction roles:
  - `/reactionrole add|remove|list`
  - Unicode emoji only
- Button roles:
  - `/buttonrole add|remove|list`
  - Click to toggle role
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

- `/ban user:<user> reason:<text?>`
- `/unban userid:<id> reason:<text?>`
- `/kick user:<user> reason:<text?>`
- `/timeout user:<user> minutes:<1-40320> reason:<text?>`
- `/purge amount:<1-100> channel:<channel?>`

Notes:
- `/unban` requires a Discord user ID, not mention or username.
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

### Channel Renewer

- `/renew add time:<HH:MM> channel:<channel?> channelname:<name?> timezone:<IANA?>`
- `/renew remove channel:<channel?> channelname:<name?>`
- `/renew list`

## Data Files

Persistent data is stored in `data/`:

- `access-control.json`
- `renew-jobs.json`
- `ticket-settings.json`
- `reaction-roles.json`
- `button-roles.json`

## Scripts

- `npm start` - Run bot
- `npm run deploy` - Register slash commands
- `npm run check` - Syntax checks
- `npm run lint` - Alias of check
- `npm run test` - Local tests
