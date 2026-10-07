---
license: cc-by-4.0
language: [en]
pretty_name: Unattended-agent failure corpus
tags: [ai-agents, llm-agents, observability, evals, reliability, agentic-ai, failure-analysis]
configs:
- config_name: default
  data_files: data/*.jsonl
---

# Unattended-agent failure corpus

**23 dated, real-world failures** from a fleet of 60+ scheduled coding-agent jobs run unattended, for months, against real money and real deadlines — not a synthetic benchmark. Each row names a **mechanism** (what the code actually did), the **consequence** (what a human reader or downstream system then wrongly believed), the **dated instance**, and the **fix or guard** that shipped (or, where one hasn't shipped yet, says so).

Spans 2026-07-31 to 2026-09-14 across 9 recurring failure classes: Cleanup blast radius, Completeness-from-partial-view, Consumption-from-incomplete-scan, Environment-assumption mismatch, Fixture-blind selfcheck, Idle daemon, Mechanism-to-consequence, Revocation-doesn't-propagate, Supervisor-kill + self-polluted probe.

**Source:** hand-curated and scrubbed from this operator's internal incident taxonomy. Nothing here is synthetic or hypothetical — every row is a postmortem of something that actually happened to a real unattended agent fleet. Latest file: `data/2026-10-06.jsonl` (23 rows).

## Four ways an unattended agent reports "done" on work it didn't do

Most rows here are some form of **false completion**: the agent, or the status surface it writes, says done / verified / live / green and a person believes it. Four classes account for most of it. Each is listed with the dated rows behind it and the guard that held.

### 1. Argued, not read (`mechanism-to-consequence`)

The agent reasons about what MUST be true and reports the inference as a checked fact. Nothing was run; the claim just sounds like something that was.

- **2026-09-07**: A plausible inference gets recorded as verified runtime truth; downstream readers treat it as checked.
- **2026-09-10**: A fleet-classification report named an LLM lane that no `claude` invocation actually backed.

**Guard:** Every "verified" claim must quote a value the agent actually read (a log line, a file, a command's output). A claim whose evidence is an argument gets flagged, not trusted.

### 2. Partial scan reported as total (`completeness-from-partial-view`)

A check walks the cases it already knows about and reports the count as exhaustive — "0 found", "all removed" — when the scope never covered the rest.

- **2026-09-01**: "0 tracked" reads as exhaustive when it only covered the paths the check's author remembered to list.
- **2026-09-07**: "0 credential shapes in any committed blob" was reported while one token sat in 16 historical blobs the scan's scope never reached.
- **2026-09-10**: One real, actively-read status document never matched the pattern and was silently excluded from the audit's "who reads this" answer.

**Guard:** A "none left" claim states its scope next to the count (working tree vs. full history, which repos, which filename pattern). No scope, no claim.

### 3. Stale green status (`revocation-doesn't-propagate`)

The status surface was set once, when the thing was wired, and never re-derived. The capability died later; the line still says live.

- **2026-09-03**: A status line claimed a phone-push integration was "armed, live in production" for ten days while every scheduled run logged the integration variable as unset and the push as skipped.
- **2026-09-11**: The job stayed "killed" in the record for a window where the fixed version had actually run zero times — the verdict and the evidence had diverged.
- **2026-09-11**: A dashboard panel reading "what's live" degrades to silently empty between CLI re-logins, with no error surfaced.
- **2026-09-12**: A silent `invalid_grant` failure looks identical to a transient network error unless something explicitly checks for it.

**Guard:** Derive status from the same run log that would record the skip or the error, and stamp a verdict with the commit it was measured against.

### 4. Green check, broken artifact (`fixture-blind-selfcheck`)

The self-test passes against a fixture it built itself, while the file the program actually ships is wrong. The green reads as evidence and nothing else is watching.

- **2026-09-13**: A green self-check sat on top of a genuinely broken shipped artifact — worse than no check, because the green reads as evidence the shipped file is sound and nothing else is watching it.
- **2026-09-13**: Re-running the exporter after the fix could never clear the bug, because "current state is correct" and "the history I export from is correct" are two separate claims, and the fix only shipped the first one.
- **2026-09-14**: A codebase can carry hundreds of green checks while most of the actual shipped outputs they're meant to protect are unchecked.

**Guard:** Any script that ships a file asserts its invariant on THAT file after the normal write path, not only on a constructed fixture.

**The common thread:** each one is a claim about the world that was never re-read from the world. The cheapest single guard across all four is to make every "done" carry the evidence it was read from (the log line, the scanned scope, the commit, the shipped file), so a claim with no evidence is visibly a claim.

The other classes in the corpus (idle daemon, supervisor-kill misdiagnosis, environment mismatch, cleanup blast radius) are covered by the rows themselves.

## License

**CC-BY-4.0.** Use it for eval sets, failure-mode taxonomies, observability-tool test fixtures, research, or anything else — attribution appreciated, not required beyond the license terms.

## Who might find this useful

Found via a search for who's currently asking these exact questions publicly:

- [Relay — a harness for AI coding agents that recover and verify](https://relayevals.com)
- [Langfuse v4 — agent evals and traces on one immutable ClickHouse table](https://langfuse.com/changelog/2026-08-17-langfuse-v4)
- [Self-bench — build SWE-bench style evals from private repos](https://github.com/mupt-ai/self-bench)
- [Traccia — observability, runtime control & audit for agents](https://traccia.ai/)
- [nearai/ironclaw issue #1993 — false-completion after a closed/reopened chat](https://github.com/nearai/ironclaw/issues/1993)
- [openai/codex issue #48348 — production outage, lies about verification](https://github.com/openai/codex/issues/48348)

No claim that any of them have seen or endorsed this — this is just where the overlap looked real.

## Fields

`date`, `class`, `class_title`, `mechanism`, `consequence`, `instance`, `fix`

## Notes

- Rows are a fixed, hand-curated snapshot, not a live scrape — this is not auto-updated from the internal ledger it was drawn from.
- Company/tool names that appear (e.g. a hosting provider's CLI token behavior, an OAuth provider's revocation semantics) describe how this operator's own integration code handled a normal product behavior — none of these rows allege a vulnerability or defect in a named third party's product.
- Parsing/classification errors in the source incidents are described as found; this is a record of failures and fixes, not a claim that the fixes are complete.
