# Reviewing one PR

## Gather

1. Metadata and description: `gh pr view <n> --repo <repo>`.
2. Full diff: `gh pr diff <n> --repo <repo>`. For very large diffs (more than ~3000
   changed lines), review the most important source files first and say in the
   report which files were only skimmed.
3. CI: `gh pr checks <n> --repo <repo>` (it is fine if there are none).
4. Surrounding code when the diff is not enough: `git show <headOid>:<path>` and
   `git grep -n <identifier> <headOid>`. Never check out the branch.

## Review

In order of importance:

- correctness bugs: logic errors, null-safety, unhandled async errors, race
  conditions, stale state, resources used after dispose, missing `mounted` checks
  after `await` in Flutter;
- security: secrets or tokens in code or logs, insecure storage, auth gaps;
- resource leaks: streams, subscriptions, timers, controllers not closed;
- error handling and failure paths;
- missing or weak tests for the changed behaviour;
- risky changes to build, CI, signing or dependency files.

Report only real issues you can tie to a line in the diff. Do not report style that a
linter covers. Do not invent issues to fill the report.

Verdict: **ready** (no blocking issue), **needs changes** (at least one critical or
high issue), or **risky** (no clear bug but large blast radius, failing CI, or
conflicts with other PRs).

## Report

One report per PR, separated by a horizontal rule:

```
## PR #<number>: <title>
```

1. **Details**: author, base branch, head commit, size (files, additions, deletions).
2. **What it does**: two or three sentences.
3. **Changes**: grouped by area of the app; one sentence per area on what changed,
   the key files, and the lines added and removed.
4. **CI and mergeability**.
5. **Issues**, most severe first. For each: severity (critical, high, medium, low),
   file and line, the failure scenario, and the suggested fix.
6. **Conflicts**:
   - with its base branch;
   - with each other PR: number, author, files and lines, with a short excerpt from
     both sides in code blocks;
   - logic warnings, if any;
   - **If this PR is merged**: which other PRs will start to conflict, and in which files.
7. **Positives**, if any, in one or two lines.
8. **Recommendation**: accept and merge, reject, or comment, with a one-sentence reason.

When this file is given to a subagent, it returns the same content as structured
notes (verdict, changes, findings with file and line, positives), and the main
session writes the report.
