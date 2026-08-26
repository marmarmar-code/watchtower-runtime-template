# Watchtower runtime template

Private runtime template for [Watchtower](https://github.com/marmarmar-code/watchtower), a deterministic monitor for public data sources.

This repository is **configuration and state only**. It contains no Watchtower program code and must never contain secrets.

## What belongs where

- Your fork of `watchtower`: program code, source adapters, tests and GitHub Actions workflow.
- Your private runtime created from this template: watchlists, search terms, filters and persisted state.
- GitHub Actions secrets in your Watchtower fork: webhook URLs, API keys and the runtime deploy key.

Each newsroom owns and operates its own fork and runtime. Changes to search terms and watchlists belong here. New technical capabilities or source adapters belong in that newsroom's Watchtower fork.

## Quick setup

### 1. Create your private runtime

Use this repository as a GitHub template and create a new repository, for example:

`your-newsroom/watchtower-runtime`

**The new runtime repository must be private.**

Do not fork this template. Use **Use this template** so your runtime starts with a clean history.

### 2. Fork Watchtower

Fork:

`marmarmar-code/watchtower`

into your own GitHub account or organisation, for example:

`your-newsroom/watchtower`

Your newsroom is responsible for that fork and any changes made to it.

### 3. Point your Watchtower fork to your private runtime

In your Watchtower fork, open:

`.github/workflows/monitor.yml`

Find the private runtime checkout and replace:

```yaml
repository: marmarmar-code/watchtower-runtime
```

with your runtime repository, for example:

```yaml
repository: your-newsroom/watchtower-runtime
```

Do not change the runtime path unless you also change the Watchtower workflow accordingly.

### 4. Give the Watchtower fork access to the private runtime

The workflow uses an SSH deploy key so the public Watchtower fork can read and update only its own private runtime.

Create one SSH key pair for this installation. Add the **public key** to the private runtime repository under:

**Settings → Deploy keys → Add deploy key**

Enable **Allow write access**, because Watchtower persists deduplication state back to `state/`.

Add the **private key** to the Watchtower fork under:

**Settings → Secrets and variables → Actions → New repository secret**

Name it exactly:

`RUNTIME_DEPLOY_KEY`

Never commit either a private key or webhook URL to a repository.

### 5. Add Microsoft Teams

Teams is the default notification provider in this template.

Create a Teams Workflow webhook for the channel that should receive Watchtower alerts. In the Watchtower fork, create this GitHub Actions secret:

`TEAMS_WEBHOOK_URL`

The URL itself must only exist as a GitHub secret.

If your newsroom uses Slack instead, change `provider = "teams"` to `provider = "slack"` in `config/watchtower.toml` and add:

`SLACK_WEBHOOK_URL`

as a GitHub Actions secret.

### 6. Optional API keys

Some source adapters require credentials. Add those as GitHub Actions secrets in the Watchtower fork, never in this runtime.

For Doffin, the expected secret is:

`DOFFIN_API_KEY`

If you do not use a source that requires an API key, you do not need to configure it.

### 7. Configure what your newsroom wants to monitor

Edit:

`config/watchtower.toml`

The included configuration contains safe placeholders. Replace `REPLACE_ME` values with your own organisations, people, topics or search terms and enable only the sources you want.

Do not put passwords, API keys, webhooks, tokens or private keys in the TOML file.

### 8. Test before enabling normal monitoring

In the **Actions** tab of your Watchtower fork, run the Watchtower workflow manually.

Recommended order:

1. `test-notification` — verify the selected Teams or Slack channel.
2. `dry-run` — verify configuration and source access without sending normal alerts or updating state.
3. `run` — establish the initial baseline.

A newly enabled source establishes a silent baseline on its first normal run. Later runs alert only on new or updated items that match your filters.

## Teams is the default

The template contains:

```toml
[notifications]
provider = "teams"
```

To use Slack instead:

```toml
[notifications]
provider = "slack"
```

Use only the corresponding webhook secret in your installation.

## Supported source types

The upstream Watchtower project currently includes adapters for:

- `regjeringen`
- `stortinget`
- `konkurransetilsynet`
- `euronext`
- `doffin`
- `hoyesterett`

Your Watchtower fork determines which technical source types are available. This runtime only determines how those available sources are configured.

## Adding something new

If Watchtower already supports the source and you only want different search terms, companies or filters, change this runtime.

If you need a completely new technical source type, implement it in **your Watchtower fork**. If the adapter is generally useful, you may optionally contribute it back to the upstream Watchtower repository as a pull request.

## Security rules

- Keep the production runtime private.
- Never commit webhook URLs, API keys, access tokens or private keys.
- Keep credentials in GitHub Actions secrets.
- Do not copy another newsroom's production runtime.
- `state/` is runtime data written by Watchtower and should normally remain tracked.
- Review configuration before making a private runtime public. Production watchlists can reveal editorial priorities even when they contain no credentials.

## Updating Watchtower

Your Watchtower fork is independent. You decide when to sync changes from `marmarmar-code/watchtower` into your fork.

If your fork contains newsroom-specific code changes, you are responsible for keeping those changes compatible when syncing upstream updates.
