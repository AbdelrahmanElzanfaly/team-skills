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

## task-flow

One task, end to end, with your approval at every gate:

1. Define the task: a short spec you approve or edit, with design images for screen changes,
   and a proposed split into smaller tasks when it is too large for one PR.
2. Upload it to your task manager (ClickUp, Jira, Linear, GitHub Issues, or none) after you approve the preview.
3. Ask before starting.
4. Build it in its own git worktree cut from your base branch, with tests, review the diff, and commit locally.
5. Ask before pushing, opening the PR and merging; then comment the PR link on the task and update your main checkout.
6. Ask before setting the task to complete.

Away mode: tell it you are away (for example "I'm away") after approving the spec, and it runs
the remaining steps without waiting, stopping only on failures, conflicts or blocked merges.

On first use in a project it asks for your task manager, where tasks go, your
statuses, base branch, worktree folder, branch naming, setup and test commands,
and merge method, and saves them to `.claude/task-flow.json`.

### Requirements

- Claude Code (terminal or the Code tab of the desktop app)
- git 2.20 or newer, and GitHub CLI signed in: `gh auth login`
- Access to your task manager: its MCP server connected in Claude Code, or its CLI

### Install

```
/plugin marketplace add AbdelrahmanElzanfaly/team-skills
/plugin install task-flow@team-skills
```

### Use

```
/task-flow:task-flow Add a password reset screen
/task-flow:task-flow setup
/task-flow:task-flow status
```
