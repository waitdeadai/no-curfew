# Contributing to no-curfew

Tiny project, low-ceremony.

## Filing an issue

If the hook fires when it shouldn't (false positive — operator legitimately asked for rest content) or fails to fire when it should (false negative — model gave unsolicited rest advice and the hook didn't catch the phrasing), please open an issue with:

1. The minimal JSON payload that reproduces the behavior. Like the fixtures in [`RECEIPTS.md`](RECEIPTS.md) — short, isolated, copy-pasteable.
2. The exit code and stderr you observed.
3. The exit code and stderr you expected.
4. Your `jq` version (`jq --version`).

## Filing a PR

PRs welcome for:

- new paternalism phrases the regex misses (false negatives)
- new allow-clause phrases the regex should respect (false positives)
- portability fixes (BSD/GNU `grep`, macOS `bash 3.2`)
- documentation improvements

Before opening:

- Add a fixture in `RECEIPTS.md` Part 1 covering the case.
- Run the new fixture against your branch and paste the exit code + stderr in the PR body.
- Keep the diff small. The hook is intentionally one short file.

## Out of scope

- Other forms of LLM paternalism (over-cautioning, refusal of helpful actions, moralizing about goals). Build a sibling hook for those if you want them.
- Disabling Claude's safety behaviors around genuine harm. The regex is intentionally narrow to the rest/wellness vocabulary cluster.
- An LLM-as-judge classifier. The whole point is bash-as-judge — non-LLM, non-rewriteable.

## Sister tools

- [no-vibes](https://github.com/waitdeadai/no-vibes) — closeout verification
- [time-anchor](https://github.com/waitdeadai/time-anchor) — local-clock injection
- [minmaxing](https://github.com/waitdeadai/minmaxing) — full governance harness

If you build a fourth textual-integrity hook on top of this pattern, send a PR adding it to the sister-tools list.
