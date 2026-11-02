# team-skills

Claude Code plugins for the team.

## pr-round

One command for the whole PR review round on a GitHub repository:

- reviews every open PR on its own, with a full report of what changed and the issues found;
- detects conflicts between PRs (who conflicts with whom, in which files and lines);
- predicts which PRs will conflict once a PR is merged;
- lets you accept and merge, reject, or comment on each PR, one at a time.

Nothing is written to GitHub without your answer for that specific PR.

### Requirements

- Claude Code (terminal, or the Code tab of the desktop app; not cloud sessions)
- GitHub CLI, signed in: `gh auth login`
- A local clone of the repository you want to review
- git 2.38 or newer

### Install

```
/plugin marketplace add AbdelrahmanElzanfaly/team-skills
/plugin install pr-round@team-skills
```

If this repository is private, you need read access to it, and git must be able to
use your GitHub credentials: run `gh auth setup-git` once.

### Use

Open Claude Code inside the repository you want to review and run:

```
/pr-round:pr-round
```

Optional arguments:

```
/pr-round:pr-round --base dev
/pr-round:pr-round --prs 31,32
/pr-round:pr-round --tests
```

### Update

```
/plugin marketplace update team-skills
```

Then update the plugin from the Installed tab of `/plugin`.
