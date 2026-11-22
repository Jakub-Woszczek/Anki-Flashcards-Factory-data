# Anki-Flashcards-Factory-data

Durable personal data for the sibling `Anki-Flashcards-Factory` code repository.
The application uses this as an ordinary directory; it never commits or pulls data.

In the **code checkout's** local `.env`, set:

```dotenv
ANKI_FACTORY_DATA_DIR=../Anki-Flashcards-Factory-data
```

Absolute paths and `~` also work. Relative paths use the code checkout, not the shell
working directory. The configured directory must exist. No other data-path setting
is needed; obsolete per-database/cache/queue variables must be removed.

## Versioned layout

| Path | Contents |
| --- | --- |
| `words_to_add/` | Live main/book queues, one entry per nonempty line |
| `anki_flashcards_factory.sqlite3` | Workspace targets/drafts, note index, save/recovery journals |
| `diki_cache/html/` | Retained raw Diki HTML snapshots, tracked in Git |
| `diki_cache/audio/` | Cached Diki MP3 recordings, tracked in Git |
| `diki_cache/index.sqlite3` | Resource digests/provenance, discovery, retry/cooldown/pacing state |
| `archives/` | Exact source backups and deliberate historical migration snapshots |
| `backups/Backup.apkg` | Historical Anki export |
| `backups/pre-data-split.sqlite3` | Application database before data-split reference rebasing |
| `backups/data-split-manifest.json` | Point-in-time SHA-256 copy inventory and reference changes |

Both active SQLite databases are deliberately versioned as binary snapshots, together
with their associated content. Stop the backend, capture tools, and background prefetch
before taking a consistent Git checkpoint or replacing data with another Git revision.
Commit the database and affected files together. Git is not a multi-machine live merge
protocol for SQLite or consuming queues.

## Local-only files

- `logs/`: downloader logs.
- `runtime/launchd/`: generated machine-specific scheduler configuration.
- `.local/`: retained local credentials, including deprecated relay configuration.
- Lock files, SQLite transient journals, temporary writes, and macOS metadata.

These are ignored. Do not add credentials or transient SQLite sidecars to a commit.
The code checkout's service credentials remain in its ignored `.env`.

The two deprecated `external_apis/tmp/` recordings were explicitly retired, not imported.
Controlled parser-regression HTML stays with the code under `tests/diki_cases/`.

## Moving or restoring

Stop writers, move/clone this directory, and update `ANKI_FACTORY_DATA_DIR`. Preserve
the database and queue/cache files as one consistent dataset. Regenerate/reinstall
the scheduler from the code checkout after changing paths:

```bash
python -m tools.diki_launchd --install
```

This installs the plist without loading it or downloading anything. See the code
handbook's `docs/features/data-storage.md` for scheduling, migration, and rollback.
The split manifest describes migration-time bytes; normal use legitimately changes
queues/databases, so it is not a permanent assertion that they stay unchanged.
