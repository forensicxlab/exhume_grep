# exhume_grep

A Rust library and CLI for searching file contents after indexing with
[`exhume_indexer`](https://github.com/forensicxlab/exhume_indexer).
Evidence and the SQLite index are opened read-only. Matches stream as they are
found, with paths and original byte offsets.

## Installation

Requires Rust 1.92 or newer.

```sh
cargo install exhume_grep --locked
```

To build from source, run `cargo build --release -p exhume_grep` from this
repository or the Exhume workspace. The binary is in `target/release/`.

## Usage

Pass the case database, evidence ID, and a search pattern:

```sh
exhume_grep -d case.sqlite --evidence-id 1 -e 'password'
```

Print matching lines with matches highlighted in red:

```sh
exhume_grep -d case.sqlite --evidence-id 1 -e 'password' --show-line
```

Color is automatic in supported terminals. Use `--color always` to force it or
`--color never` to disable it. Long lines appear as bounded excerpts.

Search a case-insensitive regex across UTF-8 and UTF-16, with NDJSON output:

```sh
exhume_grep -d case.sqlite --evidence-id 1 \
  --mode regex -e '(?i)password|secret' --encoding all --output ndjson
```

List matching files within an indexed directory:

```sh
exhume_grep -d case.sqlite --evidence-id 1 -e 'password' \
  --directory /Users --files-with-matches
```

Searches default to case-sensitive literals in UTF-8. Repeat `-e` for multiple
patterns, or use `--pattern-file indicators.txt` with one nonempty UTF-8 pattern
per line. Use `--body /new/evidence/path` if the evidence has moved.

Run `exhume_grep --help` for all options. Rust consumers can use the
[streaming API](https://docs.rs/exhume_grep).

## Search scope and results

Searches cover indexed regular files. Archive/document text extraction, deleted
files, unallocated space, and alternate streams are outside the search scope.
Regex matching is line-based; lines exceeding 8 MiB are skipped and reported as
incomplete coverage. Adjust this limit with `--max-line-bytes`.

The final summary reports whether the search completed, encountered coverage
gaps, reached a hit limit, or was cancelled. Exit codes: **0** complete with
matches, **1** complete without matches, **2** invalid, failed, incomplete, or
limited, **130** cancelled.
