# Operating Renovate Bot on GitLab

This repo illustrates how to run [Renovate Bot](https://docs.renovatebot.com/) using a GitLab CI runner, which is useful if you are on GitLab.com or a self-hosted GitLab instance, where you are obliged to run/operate the Renovate bot yourself!

It is based on the [renovate-runner](https://gitlab.com/renovate-bot/renovate-runner) GitLab.com project maintained by WhiteSource, which (IMO) applies several unsuitable defaults.

Applied changes (in relation to the [renovate-runner](https://gitlab.com/renovate-bot/renovate-runner) repo):

- Got rid of publishing *releases* for this GitLab project (it serves no purpose...)
  - Thus, there is no `.releaserc.json` or `package[-lock].json` in this repo anymore
- `renovate.json` changes:
  - Migrate to JSON**5** to be able to add comments, and to avoid that trailing commas cause problems
  - Don't extend from `"github>renovatebot/.github"` (which defines some rules which are non-sensical to us) - instead, we just extend from `"config:base"`.
  - Slim down the `packageRules` section considerably (now only covering `"renovate/renovate"`), and configure that minor and patch updates
    are *fully automatically* merged, *without* creating a PR/MR (to avoid email spam)
  - No longer pin digests (i.e., no longer put the `renovate/renovate` Docker image's SHA-256 hash in `templates/renovate.gitlab-ci.yml`). This makes no sense here, because the Renovate Bot team releases a new version (with a new version tag, of type minor or patch) about *three times per day*!! You would only use version pinning if you expect that a Docker image's version tag stays stable for a long time, but is still being rebuilt regularly.
- `.gitlab-ci.yml` changes:
  - Use `rules` instead of `only` to have more flexibility (also affects `templates/_common.gitlab-ci.yml`)
  - Make sure no pipeline is run for merge-request branches created by the bot (we don't have any tests, and CI variables are not defined for such unprotected branches)
- `templates/_common.gitlab-ci.yml` changes:
  - Require onboarding
  - Remove the `RENOVATE_EXTENDS` setting - I don't understand what it does, given that this not a "self-hosted" configuration variable, but a repository-specific variable!
  - Enable autodiscovery mode (`RENOVATE_AUTODISCOVER: 'true'`)
  - Configure the bot to create a `renovate.json5` file (instead of the default `renovate.json`) during onboarding of a developer's repository, so that the developers can use JSON comments
    
To make use of this repository, you have to:
- Create a GitLab user for the bot, e.g. `renovatebot`
- Create a personal access token for that `renovatebot` account (scopes: `read_user`, `api` and `write_repository`), and also create a personal GitHub.com access token
- In GitLab, in Settings -> CI/CD -> Variables, set the `GITHUB_COM_TOKEN` and `RENOVATE_TOKEN`
- Invite the `renovatebot` account into this project, and make sure that it has permissions to *merge* into the (usually protected) main/master branch. This can happen in different ways:
  - Option 1: invite `renovatebot` as Developer role, but then, under Settings -> Repository -> Protected branches, the main/master branch must allow "Maintainers + Developers" to do merges
  - Option 2: invite `renovatebot` as Maintainer role
- Set up a schedule (CI/CD -> Schedules) to run the CI pipeline *every hour* (cron syntax: `0 * * * *`), or however often you think it makes sense