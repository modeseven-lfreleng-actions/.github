<!--
SPDX-License-Identifier: Apache-2.0
SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# .github — Organisation-Wide Configuration

This repository contains shared configuration and community health files
for the [lfreleng-actions](https://github.com/lfreleng-actions) GitHub
organisation (Linux Foundation Release Engineering).

Files placed here are automatically inherited by all repositories in the
organisation unless overridden at the repository level.

## Contents

### Organisation Profile

- **[`profile/README.md`](profile/README.md)** — The public-facing
  organisation profile displayed at
  [github.com/lfreleng-actions](https://github.com/lfreleng-actions).
  Contains a categorised directory of all actions, tools, and test
  fixtures in the organisation.

### Shared Configuration

- **[`release-drafter.yml`](release-drafter.yml)** — Organisation-wide
  [release-drafter](https://github.com/release-drafter/release-drafter)
  configuration. Provides default categories, autolabeler rules (mapping
  Conventional Commits prefixes to labels), and version-resolver settings.
  Any repository without its own `.github/release-drafter.yml` inherits
  this configuration automatically.

### Workflows

- **[`workflows/repo-audit.yaml`](workflows/repo-audit.yaml)** — Runs on
  a weekly schedule (Monday 08:00 UTC). Compares the current list of
  repositories in the organisation against the profile README and sends a
  Slack notification to `#releng-scm` when it finds new repositories
  that lack documentation or an explicit exclusion entry.

### Repository Exclusions

- **`excluded-repos.json`** — A JSON file listing repository names to
  exclude from the weekly audit. This covers forks of upstream
  actions, placeholder/template repositories not yet developed, and backup
  directories that are not real repositories.

## How It Works

### Release Drafter Inheritance

When a repository in the `lfreleng-actions` organisation runs the
`release-drafter/release-drafter` action and does **not** have its own
`.github/release-drafter.yml`, GitHub automatically falls back to the
configuration in this repository. The shared configuration uses the
`$REPOSITORY` template variable so that release notes, download badges,
and issue links resolve to the correct URLs for any inheriting repository.

Repositories that need custom categories or version-resolver rules can
override the defaults by adding their own `.github/release-drafter.yml`.

### Repository Audit Workflow

The `repo-audit.yaml` workflow:

1. Lists all repositories in the organisation via the GitHub API
2. Filters out repositories named in `excluded-repos.json`
3. Parses the profile README for documented repository links
4. Compares the two sets and identifies any undocumented repositories
5. Posts a summary to the GitHub Actions job output
6. Sends a Slack notification to `#releng-scm` if any updates need attention

The workflow requires:

- **`SLACK_BOT_TOKEN`** — A repository secret containing a Slack bot
  token with `chat:write` permission for the
  `linuxfoundation.slack.com` workspace
- **`SLACK_CHANNEL_ID`** — A repository variable containing the channel
  ID for the `#releng-scm` channel

See the [Slack setup instructions](#slack-setup) below.

### Excluded Repositories

The `excluded-repos.json` file contains a JSON object with an
`excluded` array of repository names (not full paths) to skip
during the audit. Typical exclusions include:

- Forks of upstream actions (e.g. `gh-action-pypi-publish`)
- Repositories still at the template/placeholder stage
- Backup or archive directories

To add a new exclusion, edit `excluded-repos.json` and add the
repository name to the `excluded` array.

## Slack Setup

The repository audit workflow sends notifications using the official
[Slack GitHub Action](https://github.com/slackapi/slack-github-action).
Complete the following one-time setup steps:

### 1. Create a Slack App

1. Go to [api.slack.com/apps](https://api.slack.com/apps) and sign in
   to the `linuxfoundation.slack.com` workspace
2. Click **Create New App** → **From scratch**
3. Name it `LF RelEng GitHub Notifications` (or similar)
4. Select the `linuxfoundation` workspace

### 2. Configure Bot Permissions

1. Navigate to **OAuth & Permissions** in the app settings
2. Under **Bot Token Scopes**, add:
   - `chat:write` — to post messages
   - `chat:write.customize` — to customise the bot name/icon per message
3. Click **Install to Workspace** and authorise the app
4. Copy the **Bot User OAuth Token** (starts with `xoxb-`)

### 3. Invite the Bot to the Channel

In Slack, open the `#releng-scm` channel and run:

```text
/invite @LF RelEng GitHub Notifications
```

### 4. Get the Channel ID

1. In Slack, right-click the `#releng-scm` channel name
2. Select **View channel details**
3. The Channel ID is at the bottom of the details panel (e.g.
   `C0123456789`)

### 5. Configure GitHub Secrets and Variables

In the [`.github` repository settings](https://github.com/lfreleng-actions/.github/settings):

1. Go to **Secrets and variables** → **Actions**
2. Add a **Repository secret**:
   - Name: `SLACK_BOT_TOKEN`
   - Value: the `xoxb-` token from step 2
3. Add a **Repository variable**:
   - Name: `SLACK_CHANNEL_ID`
   - Value: the channel ID from step 4

## Contributing

Changes to this repository affect all repositories in the organisation.
Please open a pull request and ensure all pre-commit hooks pass before
merging. The repository uses the standard `lfreleng-actions` pre-commit
configuration including yamllint, actionlint, markdownlint, REUSE/SPDX
verification, and GitHub workflow schema validation.
