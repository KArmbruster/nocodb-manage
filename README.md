# NocoDB Manage — Claude Code Skill

A [Claude Code](https://claude.ai/code) skill for managing NocoDB tables, columns, views, and forms via the API. Works with any NocoDB instance (cloud or self-hosted).

## What it does

Invoke `/nocodb-manage` in Claude Code to:

- **Inspect tables** — list columns, views, field types
- **Create tables & columns** — any field type including SingleSelect with options
- **Rename** tables, views, and columns
- **Reorder columns** — in grid views and form views
- **Manage form views** — set required fields, labels, help text, heading, success message
- **Delete** columns and views (with confirmation prompts)
- **Manage records** — create, read, update, and delete rows

All operations use the NocoDB Meta API v2 with verified, tested endpoints.

## Install

Copy the skill file to your Claude Code user commands:

```bash
# macOS / Linux
cp nocodb-manage.md ~/.claude/commands/

# Windows
copy nocodb-manage.md %USERPROFILE%\.claude\commands\
```

## Configure

Add these three variables to `local.env` in your project root:

```env
NOCO_HOST=https://your-nocodb-instance.com
NOCO_BASE_ID=p1234abcd
NOCO_API_TOKEN=your-api-token-here
```

| Variable         | Where to find it                                                           |
| ---------------- | -------------------------------------------------------------------------- |
| `NOCO_HOST`      | Your NocoDB URL (e.g. `https://app.nocodb.com` or your self-hosted domain) |
| `NOCO_BASE_ID`   | URL bar when viewing a table — the `p...` segment                          |
| `NOCO_API_TOKEN` | NocoDB → Account Settings → API Tokens                                     |

## Usage

In Claude Code, type:

```
/nocodb-manage inspect table m1234abcd
/nocodb-manage add columns "Status" (SingleSelect: Yes/No) and "Due Date" (Date) to table m1234abcd
/nocodb-manage reorder columns in the default view of table m1234abcd
/nocodb-manage set all fields to required in the form view of table m1234abcd
/nocodb-manage delete the "Test Column" from table m1234abcd
```

You can also describe what you want in plain language — the skill will figure out the right API calls.

## What's inside

The skill embeds a complete, tested API reference including:

- All verified endpoints (some differ from official NocoDB docs)
- Known non-working endpoints to avoid
- Gotchas like form column properties requiring a dedicated endpoint
- Field type reference (`uidt` values)
- ID prefix guide for navigating NocoDB's identifier system

## Tested against

- NocoDB self-hosted (Railway) — v2 Meta API
- Operations verified: create/rename/delete columns, create tables, rename tables, rename/delete views, reorder in grid views, reorder in form views, set form field properties (required/label/help/description), form view settings (heading/subheading/success message), SingleSelect option management, CRUD row operations

## Notes

- The skill reads credentials from `local.env` at runtime — no secrets are stored in the skill file
- `local.env` should be in your `.gitignore`
- Rate limit: 5 requests/second per user (NocoDB enforced)
- Destructive operations always prompt for confirmation

## License

MIT — use it however you like.
