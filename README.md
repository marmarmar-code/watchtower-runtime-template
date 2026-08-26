# Watchtower runtime template

Private runtime template for [Watchtower](https://github.com/marmarmar-code/watchtower).

This repository is configuration and state only. It contains no Watchtower program code and must never contain secrets.

## Structure

- A Watchtower fork contains program code, source adapters, tests and the GitHub Actions workflow.
- A private runtime created from this template contains watchlists, filters and persisted state.
- Credentials and notification endpoints belong in GitHub Actions secrets in the Watchtower fork.

Each installation owns and operates its own fork and runtime. Configuration changes belong in the private runtime. Technical changes belong in that installation's Watchtower fork.

## Setup

### 1. Create a private runtime

Use **Use this template** to create a new private repository, for example:

`your-org/watchtower-runtime`

Do not fork this template. A template creates a clean history.

### 2. Fork Watchtower

Fork:

`marmarmar-code/watchtower`

into the account or organisation that will operate it, for example:

`your-org/watchtower`

### 3. Point the fork to the private runtime

In the Watchtower fork, open:

`.github/workflows/monitor.yml`

Replace:

```yaml
repository: marmarmar-code/watchtower-runtime
```

with the private runtime repository, for example:

```yaml
repository: your-org/watchtower-runtime
```

### 4. Configure runtime access

The workflow uses an SSH deploy key to read and update the private runtime.

Create one SSH key pair for the installation. Add the public key to the private runtime under:

**Settings → Deploy keys → Add deploy key**

Enable **Allow write access** so Watchtower can persist state.

Add the private key to the Watchtower fork under:

**Settings → Secrets and variables → Actions → New repository secret**

Name it:

`RUNTIME_DEPLOY_KEY`

Never commit private keys or webhook URLs.

### 5. Configure notifications

Microsoft Teams is the default in this template.

Add a Teams Workflow webhook to the Watchtower fork as this GitHub Actions secret:

`TEAMS_WEBHOOK_URL`

For Slack, change `provider = "teams"` to `provider = "slack"` in `config/watchtower.toml` and add:

`SLACK_WEBHOOK_URL`

### 6. Optional API keys

Some adapters require credentials. Store them as GitHub Actions secrets in the Watchtower fork, never in the runtime.

Doffin uses:

`DOFFIN_API_KEY`

### 7. Configure sources

Edit:

`config/watchtower.toml`

Replace placeholder values and enable only the required sources. Do not put passwords, API keys, webhooks, tokens or private keys in the TOML file.

### 8. Test

Run the Watchtower workflow manually from GitHub Actions.

Recommended order:

1. `test-notification`
2. `dry-run`
3. `run`

A newly enabled source establishes a silent baseline on its first normal run. Later runs alert only on new or updated items that match the configured rules.

## Notification provider

Default:

```toml
[notifications]
provider = "teams"
```

Slack:

```toml
[notifications]
provider = "slack"
```

## Source types

The upstream project includes adapters for:

- `regjeringen`
- `stortinget`
- `konkurransetilsynet`
- `euronext`
- `doffin`
- `hoyesterett`
- `brreg`

The Watchtower fork determines which source types are available. The runtime determines how those sources are configured.

## Changes

If a source is already supported and only terms, organisations, filters or options change, edit the runtime.

A new technical source type belongs in the Watchtower fork. General changes can optionally be contributed upstream.

## Security

- Keep production runtimes private.
- Never commit webhook URLs, API keys, access tokens or private keys.
- Keep credentials in GitHub Actions secrets.
- Do not copy another installation's production runtime.
- `state/` is runtime data written by Watchtower and should normally remain tracked.
- Review configuration before changing repository visibility.

## Updating Watchtower

Each fork is independent. Its operator decides when to sync upstream changes and is responsible for compatibility with any local modifications.
