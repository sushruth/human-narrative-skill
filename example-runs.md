# Example runs

A rule set that sometimes helps a lot and sometimes does nothing is hard to judge from one example. These are 30 with-and-without pairs, one run each: 4 clearly better with the skill, 21 changed a little or went both ways, and 5 came out worse. The README shows the clearest pair; this file has the rest.

## How the runs were made

Six prompts, five models, two conditions. The prompts come from outside this repo's domain: a README opener, a PR description, an explanation, a design note, a chat reply, and a commit message. Each prompt supplies its own facts.

- Claude Code: `claude -p --model sonnet` with the skill as a system prompt. User-level settings were loaded, which is why one chat reply came out in a lowercase register.
- Codex CLI 0.156.1 on its default model, with the skill text inlined above the prompt.
- `cursor-agent` in ask mode on Gemini 3.8 Flash (High), Grok 4.7 (High), and Composer 2.5, with the skill text inlined above the prompt.

The skill is `SKILL.md` as committed with this file. Outputs are unedited. The verdict column is one reader's judgment after reading both texts, and the word counts come from `wc`.

## All 30 pairs

| Prompt | Model | Words without → with | Verdict | What changed |
|---|---|---|---|---|
| README opener | Claude Code (Sonnet) | 75 → 46 | Clear | opener flips from the tool to the reader's situation; the note after the command is gone |
| README opener | Codex CLI | 35 → 36 | Clear | opener flips from the tool to the situation |
| README opener | Composer 2.5 | 135 → 44 | Small | both open on the situation; the without run adds an optional second draft |
| README opener | Gemini 3.8 Flash | 33 → 29 | Small | both open on the situation |
| README opener | Grok 4.7 | 32 → 33 | None | both open on the situation |
| PR description | Claude Code (Sonnet) | 141 → 141 | Small | the reason moves to the top |
| PR description | Codex CLI | 96 → 117 | Mixed | adds "Test execution is unverified," a disclaimer nobody asked for |
| PR description | Composer 2.5 | 177 → 214 | Mixed | longer, with more sections |
| PR description | Gemini 3.8 Flash | 120 → 177 | Mixed | the without run already led with the symptom; the with run is longer |
| PR description | Grok 4.7 | 149 → 143 | None | same layout in both |
| Explanation | Claude Code (Sonnet) | 235 → 234 | Small | both clear; the with run gives a concrete read count |
| Explanation | Codex CLI | 161 → 151 | Small | similar |
| Explanation | Composer 2.5 | 191 → 236 | Mixed | em-dashes gone, but it opens by telling the reader what they are doing |
| Explanation | Gemini 3.8 Flash | 153 → 155 | Clear | textbook analogy and em-dash gone; drops the closing trade-off sentence |
| Explanation | Grok 4.7 | 133 → 150 | Small | similar |
| Design note | Claude Code (Sonnet) | 444 → 303 | Clear | fewer invented specifics, one open question named |
| Design note | Codex CLI | 171 → 169 | Small | similar |
| Design note | Composer 2.5 | 340 → 535 | Worse | longer, with extra sections, em-dashes, and negation shapes |
| Design note | Gemini 3.8 Flash | 201 → 132 | Small | similar |
| Design note | Grok 4.7 | 221 → 186 | Mixed | opens on the price quote, then "decline the $140 Postgres quote" |
| Chat reply | Claude Code (Sonnet) | 74 → 59 | Small | same verdict first; lowercase chat register from user-level settings |
| Chat reply | Codex CLI | 48 → 43 | Worse | a situation line lands before the verdict |
| Chat reply | Composer 2.5 | 94 → 46 | Worse | a situation line lands before the verdict |
| Chat reply | Gemini 3.8 Flash | 43 → 73 | Worse | a situation line lands before the verdict |
| Chat reply | Grok 4.7 | 42 → 52 | None | similar |
| Commit message | Claude Code (Sonnet) | 54 → 47 | Small | the reason moves after the change |
| Commit message | Codex CLI | 31 → 46 | Small | similar |
| Commit message | Composer 2.5 | 114 → 101 | Worse | a status sentence lands before the commit message |
| Commit message | Gemini 3.8 Flash | 47 → 47 | None | word for word the same |
| Commit message | Grok 4.7 | 35 → 46 | None | similar |

Verdicts: Clear means the with run is better on the point the skill targets. Small means the two read alike with minor differences. None means the same or nearly the same. Mixed means better on one axis and worse on another. Worse means the with run reads worse than the without run.

## Where it helped

### README opener, Claude Code (Sonnet)

The without run opens on what the tool is and ends with a note about what the writer left out. The with run opens on the reader's situation, gives the same facts, and stops after the command. This is the pair in the README.

**Prompt:** Write the opening of the README for `tzsplit`, a CLI that reads a CSV of timestamps and splits it into one file per local calendar day, using each row's own timezone column. Two or three sentences, then the install command (`cargo install tzsplit`).

Without the skill:

`````
```markdown
# tzsplit

`tzsplit` reads a CSV of timestamps and splits it into one file per local calendar day. Each row is bucketed using its own timezone column, so a row stamped 23:30 in `America/Los_Angeles` lands on that Pacific day, not the UTC one.

    cargo install tzsplit
```

I didn't state the timezone column's format (IANA names vs. offsets), the output filenames, or the flags, because I don't know them. Add those once they're settled.
`````

With the skill:

`````
```markdown
# tzsplit

Each row in a multi-timezone CSV reaches midnight at a different moment. `tzsplit` reads a CSV of timestamps and writes one file per local calendar day, using each row's own timezone column to decide which day it belongs to.

    cargo install tzsplit
```
`````

### Design note, Claude Code (Sonnet)

The without run adds WAL mode, an online-backup command, S3 versioning, a "Not chosen" section, and four revisit conditions that the prompt never gave. The with run keeps to the given facts and names one open question, whether clients cache their last known values.

**Prompt:** Write a short design note recommending that our internal feature-flag service store its data in SQLite on a single node instead of a managed Postgres. Facts: about 400 flags, roughly 50 reads per second at peak, writes a few times a day from one admin UI, a nightly backup to S3 is acceptable, a restart takes about 3 seconds, the team has no Postgres on-call rotation, and the managed Postgres quote is $140 a month.

Without the skill:

`````
# Design note: SQLite for the feature-flag service

**Recommendation:** store flags in SQLite on the service's single node. Skip the managed Postgres.

## Why

- **Load is tiny.** 400 flags at about 50 reads/s peak is far below what one SQLite file handles. Writes are a few per day from one admin UI, so there is no write contention.
- **No Postgres on-call.** Managed Postgres still leaves us owning failover behavior, connection limits, upgrades and incident response, with nobody on rotation for it. SQLite removes that surface. It is one file next to the process.
- **Cost.** The Postgres quote is $140/month, about $1,680/year. SQLite adds no line item beyond the node we already run and the S3 storage for backups, which is negligible for a file this size.
- **Restart is short.** About 3 seconds of downtime on restart is acceptable for this service. (Assumes SDK clients cache flag values and fall back to defaults. See Risks.)

## Design

1. One SQLite file on the node's persistent disk, in WAL mode.
2. The service is the only process that opens it.
3. Nightly job: use SQLite's online backup (`sqlite3 flags.db ".backup ..."`, not a plain `cp`), then upload to S3 with versioning on.
4. Restore procedure: fetch the latest backup, drop it in place, restart. Test it once before launch and write the steps in the runbook.

## Risks and accepted tradeoffs

- **Single node is a single point of failure.** If the node or disk dies, flag reads are down until we restore or rebuild. This is the main cost of the choice. The mitigation is client-side caching with safe defaults, which I have not verified that our SDKs do. Check that first.
- **Up to 24h of admin edits can be lost** between nightly backups. With a few writes per day from one UI, the loss is a few edits to redo. You stated that nightly backup is acceptable, so I am treating this as accepted.
- **No read replicas or multi-region reads.** Not needed at this load.

## Not chosen

- **Managed Postgres.** Better availability and point-in-time recovery, but the cost and on-call burden are out of proportion to 400 flags.
- **SQLite plus continuous replication (e.g. Litestream).** Would shrink data loss from a day to seconds. Not adopted now, and it is the obvious upgrade if the nightly gap ever becomes a problem.

## Revisit if

- Flags become a hard dependency of the request path with no client-side fallback.
- We need more than one writer, or more than one service node.
- Losing a day of edits becomes unacceptable.
- Read load grows by orders of magnitude.
`````

With the skill:

`````
# Design note: SQLite for the feature-flag service

Our flag service holds about 400 flags and is written to a few times a day, so a database server is more than it needs. **Recommendation: store flags in SQLite on a single node, with a nightly backup to S3, and skip the managed Postgres.**

## Why

- **Load:** 50 reads per second at peak over 400 rows is well within what SQLite serves from one process.
- **Writes:** They come from one admin UI, a few times a day, so SQLite's single-writer limit never comes up.
- **Cost:** Managed Postgres is quoted at $140 a month, $1,680 a year. A single node running SQLite adds no database line item.
- **Ownership:** The team has no Postgres on-call rotation, so a Postgres incident would have no one with the context to handle it. A SQLite file is a single file the team can copy and restore.

## What we accept

- **Downtime on restart:** The service is unavailable for about 3 seconds each time it restarts.
- **Data loss window:** With a nightly backup, a node failure can lose up to a day of flag edits. At a few writes a day, those edits can be re-entered from the admin UI.
- **No failover:** One node means one point of failure. Recovery is launching a new node and restoring the latest S3 backup.

## Open question

I don't know whether the flag clients cache their last known values. If they do, a 3-second restart is invisible to them. If they don't, they see errors for those 3 seconds, and that decides whether this is acceptable.

## Revisit if

Writes move beyond the single admin UI, the flag count or read rate grows by an order of magnitude, or a day of lost edits becomes unacceptable.
`````

### Explanation, Gemini 3.8 Flash

The without run opens on a textbook analogy and uses an em-dash. The with run opens on the situation and has none. It also ends on the page-split mechanism and drops the closing trade-off sentence.

**Prompt:** Explain to a new backend engineer why adding an index to a table speeds up reads and slows down writes. Plain prose, one or two paragraphs.

Without the skill:

`````
Think of a database index like the index at the back of a textbook. Without one, the database has to perform a full table scan—reading every single row from disk to find what you're looking for. An index creates a separate, pre-sorted data structure (typically a B-tree) that maps column values directly to the physical storage locations of their rows. When you query an indexed column, the database can traverse this balanced tree in logarithmic time rather than scanning millions of rows sequentially, drastically reducing disk I/O and query latency.

The trade-off is that every index introduces overhead on write operations (`INSERT`, `UPDATE`, and `DELETE`). Whenever data changes, the database cannot simply write the new row and finish; it must also synchronously update and rebalance each index that references that table. More indexes mean more auxiliary structures to write and keep sorted, which increases write amplification, locks resources longer, and slows down throughput.
`````

With the skill:

`````
Table queries slow down as rows accumulate. An index cuts query time to a few steps while adding extra work to every write. Querying the table directly requires scanning disk pages in arrival order to evaluate every row. An index replaces that scan with a search tree of sorted column values, paired with pointers to each row's disk page. Searching the tree lets the database jump through branch nodes by value, resolving the lookup in three or four page reads.

Writes slow down because every insert, update, and delete must modify both the table and the search tree. An insert writes the new row to the table storage page, then traverses the search tree to place the indexed column value into sorted position. That placement adds memory comparisons and disk writes for each indexed column. When a tree page fills with entries, the database splits the page and updates parent nodes before finishing the write.
`````

## Where it changed little

### Commit message, Gemini 3.8 Flash

The two texts are word for word the same.

**Prompt:** Write the commit message for this change: `retry_on_timeout` in `client/http.py` used to retry every request 3 times; it now retries only idempotent methods (GET, HEAD, PUT, DELETE) and leaves POST alone, because a retried POST /payments created duplicate charges in two incidents last month. Tests updated in `tests/test_http_retry.py`.

Without the skill:

`````
```text
Retry only idempotent methods in retry_on_timeout

retry_on_timeout in client/http.py used to retry every request 3 times.
It now retries GET, HEAD, PUT, and DELETE, and sends POST once.

A retried POST /payments created duplicate charges in two incidents
last month.

tests/test_http_retry.py is updated to match.
```
`````

With the skill:

`````
```text
Retry only idempotent methods in retry_on_timeout

retry_on_timeout in client/http.py used to retry every request 3 times.
It now retries GET, HEAD, PUT, and DELETE, and sends POST once.

A retried POST /payments created duplicate charges in two incidents
last month.

tests/test_http_retry.py is updated to match.
```
`````

### PR description, Grok 4.7

Both use the same two sections and the same table. The with run adds the 02:00 detail.

**Prompt:** Write the description for this pull request.

Changes: the nightly invoice export job now writes to a temporary file and renames it into place when finished. Before, it wrote straight to `exports/invoices.csv`, and the finance team's importer sometimes picked up a half-written file at 02:00. Touched files: `jobs/export_invoices.py` (new `write_atomic` helper, 18 lines), `tests/test_export_invoices.py` (one new test that kills the job mid-write and checks the old file is intact). The rename is atomic only on the same filesystem; `/tmp` is a different mount in production, so the temp file is created next to the target. Not changed: the importer, the schedule, the CSV format. Risk: low; the one thing to check is that the `exports/` directory is writable by the job user, which it already is today.

Without the skill:

`````
## What and why

- The nightly export sometimes left finance a half-written invoice file. The job now renames a finished temp file over the destination.

## How to review

| File | What to check |
| --- | --- |
| `export_invoices.py` | The temp file is created next to the destination and replaces it only after the write finishes. |
| `test_export_invoices.py` | Killing the job mid-write leaves the previous export intact. |

## Details

- A rename is atomic only when the temp file and the destination are on the same filesystem. The temp file is created next to the destination because production `/tmp` is a different mount.
- The importer, the export schedule, and the CSV format stay the same.
- The temp file is created in the export directory the job already writes, and that directory is writable by the job user today.
`````

With the skill:

`````
## What and why

- Finance sometimes imported a half-written invoice file at 02:00. The nightly export now writes the invoice file aside and renames it into place when finished.

## How to review

| File | What to check |
| --- | --- |
| `export_invoices.py` | The new helper writes a temporary file in `exports/`, then renames it onto the invoice file. |
| `test_export_invoices.py` | The new test kills the export mid-write and checks the previous invoice file is intact. |

## Details

- `/tmp` is a different mount in production. A rename between that mount and `exports/` is not atomic. The temporary file is created in `exports/`.
- The importer, the 02:00 schedule, and the CSV format stay as they are.
- The job user can already write `exports/`, the directory the finished invoice file is renamed in.
`````

## Where it got worse

The skill's first-line rule is about documents. In a chat reply or a commit message it can put a situation sentence in front of the verdict or the subject line, which is the opposite of what a reader of those wants first.

### Chat reply, Gemini 3.8 Flash

The with run puts "Your pull request is waiting on a red CI test." ahead of the answer. Codex and Composer did the same.

**Prompt:** A teammate asked in chat whether they can merge their PR today. Facts: CI is red on one test (`test_refund_rounding`), which also fails on main since yesterday's dependency bump; the PR itself touches only docs; the owner of the failing test is on leave until Monday; merges to main are allowed with a red check if a maintainer approves in the PR. You are a maintainer. Write your reply to them.

Without the skill:

`````
Yes, you can merge today. I'll approve the PR now.

Your PR only touches docs, and `test_refund_rounding` has been failing on main since yesterday's dependency bump. The test owner is out until Monday, so we can follow up with them when they're back.
`````

With the skill:

`````
Your pull request is waiting on a red CI test.
You can merge today. Merges to main with a red check are allowed when a maintainer approves. The changes modify documentation files, so I approved the pull request.

The failing test is `test_refund_rounding`, which has failed on the main branch since yesterday's dependency bump. The owner of `test_refund_rounding` is on leave until Monday, so the failure remains on the main branch until then.
`````

### Commit message, Composer 2.5

The with run puts a status sentence before the commit message.

**Prompt:** Write the commit message for this change: `retry_on_timeout` in `client/http.py` used to retry every request 3 times; it now retries only idempotent methods (GET, HEAD, PUT, DELETE) and leaves POST alone, because a retried POST /payments created duplicate charges in two incidents last month. Tests updated in `tests/test_http_retry.py`.

Without the skill:

`````
Suggested commit message:

```
Stop retrying POST in retry_on_timeout

retry_on_timeout in client/http.py retried every request three times.
Retried POST /payments calls caused duplicate charges in two incidents
last month. Timeouts on GET, HEAD, PUT, and DELETE still retry; POST
does not.

Update tests in tests/test_http_retry.py.
```

**Subject:** States the behavioral change readers care about (POST no longer retried) instead of only naming the helper.

**Body:** Incident context first, then the exact method split, then the test file—aligned with a why-first message and the facts you gave.

Shorter subject if your repo prefers the helper in the title:

```
Retry only idempotent methods in retry_on_timeout
```

Keep the same body; only the subject line changes.
`````

With the skill:

`````
The HTTP client was retrying POST on timeout the same as safe methods.

Use this commit message:

```
http: retry timeouts only for idempotent methods

retry_on_timeout in client/http.py retries GET, HEAD, PUT, and DELETE up to
three times after a timeout. POST does not retry. Duplicate charges on
POST /payments followed retried timeouts in two incidents last month.
tests/test_http_retry.py updated.
```

The subject is the one-line change for `git log`. The body states the new behavior, the POST limit, the reason, and the test file. The duplicate-charge detail comes from your briefing; the workspace here had no `client/http.py` to verify against.
`````

### Design note, Composer 2.5

The with run is 195 words longer, adds "Ask" and "Review focus" sections, and has three em-dashes and three negation shapes by a rough count.

**Prompt:** Write a short design note recommending that our internal feature-flag service store its data in SQLite on a single node instead of a managed Postgres. Facts: about 400 flags, roughly 50 reads per second at peak, writes a few times a day from one admin UI, a nightly backup to S3 is acceptable, a restart takes about 3 seconds, the team has no Postgres on-call rotation, and the managed Postgres quote is $140 a month.

Without the skill:

`````
# Feature flags: store state in SQLite on one node

**Recommendation:** Run the internal feature-flag service on a single node with SQLite as the datastore. Skip managed Postgres at ~$140/month unless requirements change.

## Workload

- ~400 flags
- ~50 reads/sec at peak (in-memory cache can sit in front of the DB)
- Writes a few times per day from one admin UI
- Nightly backup to S3 is acceptable
- Process restart ~3 seconds

## Why SQLite fits

SQLite handles this read/write profile easily. The dataset is small (hundreds of rows), writes are rare and single-writer, and peak read rate is low for a local file DB when the hot path is cached.

Ops stay on the team’s existing muscle memory: one binary (or container), one file on disk, no separate database tier. There is no Postgres on-call rotation today; managed Postgres adds a second system to monitor, patch, and debug for little gain at this scale.

Cost: ~$140/month for managed Postgres vs. marginal cost of SQLite on hardware you already run.

## How it would run

- Single active node holds the SQLite file; reads served from process memory after load or periodic refresh.
- Admin UI writes go through the same node (or a short write path that still lands on that file).
- Nightly snapshot of the DB file (or `.backup`) to S3; restore = replace file and restart (~3s).

## Caveats

- **Availability:** One node means planned or unplanned downtime affects flag reads until restart or failover. Acceptable if clients tolerate stale flags briefly or you can fail open/closed per flag.
- **HA:** No built-in multi-writer replication like Postgres. A second warm standby with manual or scripted promotion is possible later; not required for current traffic.
- **Growth:** If flags, write rate, or multi-region/active-active requirements jump materially, revisit Postgres or another shared store.

## Decision

Adopt SQLite on a single node for the feature-flag service given current scale, backup policy, and ops constraints. Re-evaluate when write frequency, HA needs, or data size outgrow single-file SQLite.
`````

With the skill:

`````
The feature-flag service is choosing where its state lives.

**Recommendation:** run one node with SQLite; skip managed Postgres for this workload.

## Ask

Approve storing feature-flag data in SQLite on a single application node, with nightly backup to object storage (S3), instead of provisioning managed Postgres at about $140/month.

## Load and shape

Roughly 400 flags, about 50 read requests per second at peak, and a handful of writes per day from one admin UI. Reads dominate; writes are rare and single-writer in practice. A process restart that reloads state takes on the order of three seconds. That profile fits a local embedded database on one host more closely than it fits a separate always-on relational service.

## Why SQLite fits here

SQLite holds the full dataset in one file on the same machine as the app. Peak read traffic at this scale is well within what a single SQLite connection pool on modest hardware can serve when queries are simple key lookups. No network hop to a database tier on every read; latency stays predictable for flag resolution.

Operational surface stays small. The team has no Postgres on-call rotation. Managed Postgres adds a second system to patch, monitor, connection limits, and incident response for a service whose writes are occasional admin edits. SQLite’s failure modes on one node are mostly “disk full,” “corrupt file,” or “host down”—the same host the app already depends on.

Cost: managed Postgres at about $140/month buys availability and ops the workload does not need at this read/write ratio. SQLite’s incremental cost is disk on the existing node and the backup pipeline you already accept.

## What you give up

Single-node SQLite is not multi-writer HA. If the node dies, flags are unavailable until the host or process is back, unless you run a cold standby or restore from backup. Concurrent writes from many clients would contend; your model (few writes, one UI) avoids that. Horizontal scale-out of flag readers usually means caching or replicas elsewhere, not sharding the flag store—out of scope unless traffic grows an order of magnitude or more.

Nightly backup to S3 means recovery point objective is up to one day unless you add more frequent snapshots or WAL shipping later. For internal flags where stale values for hours are a known tradeoff, that is often acceptable; confirm with product for kill-switch and compliance flags.

## How it would run (sketch)

One deployment (or a small fixed pair with one active writer if you need a warm spare). Application opens SQLite with WAL mode for read concurrency during writes. Admin UI writes go through the app with transactions. Nightly job copies the database file (or uses SQLite backup API) to S3; document restore: download file, place on node, restart (~3 seconds).

## Open checks before merge

- Confirm no requirement for sub-minute RPO or cross-region active-active.
- Confirm peak 50 rps is sustained headroom, not a spike that will 10× soon.
- Decide whether any flag is regulated or safety-critical enough to warrant managed DB audit logs from day one.

## Review focus

Look at backup/restore runbook, single-writer assumption in the admin path, and monitoring on disk usage and backup success—not connection pooling or replication design.
`````

## Other things the full set shows

- Codex added "Test execution is unverified" and "performance on the intended node remains unverified" to the PR description and the design note. The prompts asked for neither.
- Composer opened the explanation with "You're wiring queries against a large table," a statement about the reader that the prompt did not support.
- Gemini and Grok already opened the README on the situation without the skill. The opener rule changed Claude and Codex.
- Grok's outputs barely moved on any prompt.
