# Conflicts between PRs and merge impact

GitHub only checks each PR against its own base. Two PRs can both be clean against
`dev` and still conflict with each other. Everything here works on commit ids and
writes nothing but git objects: no branch, index or file is changed, and nothing
needs cleaning up.

Notation: `B` is `origin/<base>`, `A` and `C` are PR head commit ids (`headRefOid`).
Only compare PRs that share the same base.

## 1. Changed files per PR

```bash
git diff --name-status -M $(git merge-base B A) A
```

Record `mergeable == CONFLICTING` PRs: they already conflict with their base. For
those, list the files with:

```bash
git merge-tree --write-tree --name-only B A
```

Build the overlap list: every pair of PRs that touch at least one common file,
including renames and deletes. Pairs with no common file cannot conflict in git.

## 2. Git conflicts between two PRs

For each overlapping pair A, C:

```bash
git merge-tree --write-tree --name-only A C
```

- Exit code 0: no textual conflict.
- Exit code 1: conflict. Line 1 is a tree id, then the conflicted paths, then an
  empty line, then messages such as `CONFLICT (content): Merge conflict in <path>`
  that give the kind (content, modify/delete, rename/rename, add/add).

For each conflicted path, find the clashing regions:

```bash
git show <tree id>:<path> | grep -n -E '^(<<<<<<<|=======|>>>>>>>)'
```

and the lines each PR changed in that file:

```bash
git diff -U0 $(git merge-base B A) A -- <path>
git diff -U0 $(git merge-base B C) C -- <path>
```

Keep a short excerpt (at most ~15 lines per region) of what each side wrote.

Lock and generated files (`pubspec.lock`, `Podfile.lock`, `*.g.dart`,
`*.freezed.dart`, `project.pbxproj`, `Package.resolved`) do conflict in git; report
them, but mark them as usually resolved by regenerating rather than by hand.

## 3. Logic warnings (no git conflict, still risky)

For pairs that merge cleanly, read both diffs and look for:

- one PR renames, removes or changes the signature of a class, function, provider,
  route, enum value or JSON field that the other PR adds new uses of;
- both PRs change the same function, provider, model or constant in different places;
- both PRs add the same key, route name, enum value or migration;
- one PR changes an API endpoint, model shape or permission check that the other
  PR's new code relies on.

Confirm a use really exists with `git grep -n <identifier> C`. Report these as
warnings with a confidence level, not as conflicts. Do not warn just because both
PRs touch the same file.

If `--tests` was passed, or the user asks, verify the riskiest pair: create a
temporary worktree in the scratchpad directory from `B`, merge A then C there with
`--no-commit`, run the project's tests (Flutter: `flutter pub get` then
`flutter test`), report the result, then `git worktree remove --force` it. Never do
this in the user's own checkout.

## 4. Merge impact: "if A is merged, who breaks?"

Simulate the base after merging A, as an unreferenced commit:

```bash
T=$(git merge-tree --write-tree B A)        # only when A merges cleanly into B
S=$(git commit-tree "$T" -p B -p A -m "simulated merge of PR <n>")
```

Then for every other PR C on the same base:

```bash
git merge-tree --write-tree --name-only S C
```

Exit code 1 means: once A is merged, C will conflict, in the listed files. Collect
the regions as in section 2. This is the precise answer to "if I accept A, will
C conflict, and where". `S` is not referenced by any branch and is harmless.

## 5. Merge order

Per base, suggest an order that minimises rework:

- PRs that overlap with nobody can be merged any time.
- In a conflicting pair, prefer first the PR that is smaller, already approved, or
  that the other depends on; the other author then updates their branch.
- Use section 4 to pick the order with the fewest conflicts left over.
- State who has to resolve what after each merge.

## 6. After a real merge

Run `git fetch --no-tags origin <base>` and, for each remaining PR C on that base:

```bash
git merge-tree --write-tree --name-only origin/<base> C
```

Report the PRs that now conflict, with files and regions, and adjust the order.
