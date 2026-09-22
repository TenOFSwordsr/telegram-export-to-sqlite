# Project XHTTP Telegram Export Database

A parser that turns a 594-page iMe/Telegram Desktop HTML export of the "Project XHTTP" group
(`projectXhttp`, an anti-censorship proxy community) into a queryable SQLite database with an FTS5
text index, plus `NOTES.md` recording what was found in it. Nearly half a million messages spanning
2025-02-01 to 2026-09-10, so the value here is the schema and the HTML quirks handled, not the data.

**Suggested repo name:** `telegram-export-to-sqlite`
**Stack:** Python 3 stdlib only (`html.parser`, `sqlite3`, `re`), SQLite with FTS5
**Status:** finished
**Last modified:** 2026-09-09

## What it does

- `parse_export.py` - walks `messages*.html` in the export directory in numeric order and feeds each
  file to a single-pass `HTMLParser` subclass that recognises `message default` / `message service`
  divs, `from_name`, `pull_right date details` (full timestamp taken from the `title` attribute),
  `reply_to details` (href `#go_to_messageN`), `forwarded body` (author plus embedded date span),
  `signature details`, `bot_button`, and the `media_*` wraps for photo / video / file /
  audio_file / voice_message / poll including poll question, answers and total.
  Consecutive messages from one sender arrive as `joined` rows with no name, so the last seen sender
  is carried forward to keep attribution at 100%. Dates are normalised to UTC ISO, human sizes
  ("12.4 MB") become `size_bytes`, and `messages_fts` is rebuilt at the end.
  Prints row-count verification for total / user / service / with-text / media / unique senders.
- `xhttp.sqlite` - the generated database (~98 MB): `messages`, `senders`, `media`, `bot_buttons`,
  `messages_fts`.
- `NOTES.md` - chat identity, schema documentation, counts (473,393 rows, 471,603 user messages,
  11,986 unique senders, 339,667 replies, 133,877 replies pointing outside the export window),
  monthly activity shape, top keyword counts, the tools the community discussed, and known gotchas.

## Layout

```
parse_export.py   HTML export -> SQLite (whole program, ~380 lines)
NOTES.md          schema reference and analysis findings
xhttp.sqlite      generated database (do not publish)
```

## Running it

```bash
python parse_export.py [export_dir] [db_path]
```

Both arguments default to the author's own paths (the export under `Downloads\iMe Desktop`, the DB
in this folder). The script deletes and rebuilds all four tables on every run.

## Notes

- Publishing blocker: `xhttp.sqlite` is 471k messages with ~12k display names from a real group -
  other people's personal data, not your code. Delete the database (and the derived statistics in
  `NOTES.md` if you want to be careful) before making this repo public. It is also ~98 MB, close to
  GitHub's 100 MB per-file ceiling, so it does not belong in git regardless.
- Media binaries were never in the export; every `media` row carries a name, status string and a
  size that is only parseable when the file was included.
- `NOTES.md` documents that 15,098 unique bare hrefs in message text are mostly config endpoints
  typed into chat - another reason not to ship the database.
- `__pycache__/` is generated output.
