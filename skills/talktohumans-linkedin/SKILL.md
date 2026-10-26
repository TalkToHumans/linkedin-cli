---
name: talktohumans-linkedin
description: Use the tth CLI (TalkToHumans LinkedIn CLI) to work with the user's LinkedIn from the terminal - prospecting, finding and filtering contacts and companies, importing and enriching profiles, reading DMs and conversations, tagging, writing message drafts and sequences the user reviews and sends, and LinkedIn activity reports.
---

# TalkToHumans LinkedIn CLI (`tth`)

`tth` reads a synced copy of the user's LinkedIn workspace in TalkToHumans and writes drafts. It never sends a LinkedIn message: drafts and sequences wait for the user to review and send them in TalkToHumans.

## Start here

1. Run `tth skills show ground-rules` once per session and follow it for every task.
2. Run `tth skills list`; when the request matches a playbook, run `tth skills show <name>` and follow its steps.
3. Run `tth <command> --help` before using a command for the first time: examples come first, then flags.

## Auth

- `tth auth status` shows the workspace and credential. Exit code `4` means not logged in, plan, or setup.
- People log in with `tth auth login` (browser). Scripts and cron set `TTH_API_KEY` (an admin generates it in Settings; a new key revokes the previous one).
- A setup error (`workspace_setup_required`, `linkedin_not_connected`) is finished in the web app the error links, not by logging in again.

## Playbooks name MCP tools

The playbooks are shared with the TalkToHumans MCP. Translate each tool to its command: `verb_noun` becomes `tth <nouns> <verb>`, input fields become kebab-case flags (`account_id` → `--account-id`) except the ID of the record a command acts on, which is its argument (`update_contact` `contact_id` → `tth contacts update <contact-id>`), list inputs become repeatable flags (`account_ids` → `--account-id 1 --account-id 2`), and lists take `--all`.

| MCP tool | Command |
|---|---|
| `list_accounts` | `tth accounts list` |
| `get_credit_balance` | `tth credits show` |
| `list_contacts` | `tth contacts list` |
| `import_contacts` / `get_import_progress` | `tth contacts import <urls…>` / `tth contacts import-status <import-id>` (both take `--wait`) |
| `update_contact` | `tth contacts update <contact-id>` |
| `enrich_profile` | `tth contacts enrich` |
| `find_email` / `find_phone` | `tth contacts find-email <contact-id>` / `find-phone <contact-id>` |
| `list_companies` | `tth companies list` |
| `list_field_definitions` / `create_field_definition` | `tth fields list` / `create` |
| `list_tags` / `update_tag` / `delete_tag` | `tth tags list` / `update <tag-id>` / `delete <tag-id>` |
| `list_conversations` | `tth conversations list` |
| `list_messages` | `tth messages list <conversation-id>` |
| `write_message_draft` | `tth drafts write` |
| `update_sequence` / `delete_sequence` | `tth sequences update <sequence-id>` / `delete <sequence-id>` |
| `list_templates` / `create_template` / `update_template` | `tth templates list` / `create` / `update <template-id>` |
| `list_views` / `create_view` / `update_view` | `tth views list` / `create` / `update <view-id>` |
| `list_activity` / `get_activity_summary` | `tth activity list` / `summary` |
| `load_skill` | `tth skills show <name>` |
| `send_feedback` | `tth feedback send` |

Any other `/v1` route: `tth api <METHOD> /v1/... [--field k=v] [--input file] [--all]`.

## Output contract

- stdout is data only: JSON when piped (always, for agents), a table in a terminal, `--json` forces JSON. A single page prints the API response unchanged (`data.pagination.next_cursor` → `--cursor`); `--all` prints one JSON array of every item.
- stderr carries warnings and errors. Errors are JSON when piped: `{"code","message","hint"}`. Read sync and caveat warnings and tell the user when data may be incomplete.
- Exit codes: `0` ok, `1` error, `2` usage (fix the command), `3` not found, `4` auth, plan or setup, `5` rate limited.
- `tth tags delete` and `tth sequences delete` need `--yes` without a terminal: pass it only when the user named that tag or sequence, or agreed.
- Imports, enrichment and email or phone lookups use credits and don't ask.

## Rules

- Drafts only: `tth drafts write` saves a draft or stages a sequence. Never say a message or request was sent; point the user to the Drafts view.
- When a command fails or returns something wrong, the user wants something no command can do, or help was confusing, run `tth feedback send --kind bug|missing|confusing|praise --message "..."` (no names, URLs, emails or message text).
