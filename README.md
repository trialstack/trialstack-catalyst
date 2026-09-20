# TrialStack Catalyst plugin distribution

TrialStack is the clinical development system for teams and agents. TrialStack
Catalyst is its governed agent interface: the assistant discovers and reviews
governed trial data, prepares each change with a complete preview, and
executes a permitted update only after explicit approval in a follow-up
message, then reads the result back. Prepared intents expire after ten
minutes and cannot be replayed.

This repository is the shared public distribution for Codex and Grok Build, with a Grok Bot candidate.
It contains the plugin manifests, the hosted MCP connector config, and the
canonical governed skill. It contains no second runtime and no second skill
copy: every client uses the same production endpoint and the same workflow.

## Contents

- `.cursor-plugin/marketplace.json` — Cursor/Grok Bot submission index pointing to the nested Catalyst plugin.

- `plugins/trialstack-catalyst/.codex-plugin/plugin.json` — Codex plugin
  identity, starter prompts, and skill/MCP pointers.
- `plugins/trialstack-catalyst/.grok-plugin/plugin.json` — Grok plugin
  manifest pointing at the same skill and MCP config.
- `plugins/trialstack-catalyst/.mcp.json` — connects to the hosted Catalyst
  MCP endpoint (see Endpoint and auth below).
- `plugins/trialstack-catalyst/skills/trialstack-catalyst/SKILL.md` — the
  canonical governed workflow: read-only discovery, organization selection,
  and two-step prepare/approve/execute changes.
- `plugins/trialstack-catalyst/assets/catalyst-icon.svg` — brand icon
  referenced by the Codex manifest.
- `.agents/plugins/marketplace.json` — Codex marketplace index for the
  `trialstack` marketplace.
- `.grok-plugin/marketplace.json` — Grok marketplace index for local and
  git-based marketplace adds.

## Endpoint and auth

The plugin connects only to `https://api.trialstack.com/mcp`. Sign-in uses
the server's OAuth flow; every organization-scoped call passes an explicit
server-returned `organizationId`, and Catalyst verifies membership, plan
access, and current TrialStack capabilities on each call. The plugin reads
no local credentials, `.env` files, or unrelated filesystem data, and it
installs no local binary.

OAuth status by client:

- Codex: complete the TrialStack OAuth flow after installation (verified
  path).
- Grok Build: MCP authorization behavior for this server has not been
  verified yet. Do not assume one-click OAuth works; verify sign-in and
  `list_organizations` before relying on it.

## Grok Bot candidate

Version 0.5.0 adds `plugins/trialstack-catalyst/.cursor-plugin/plugin.json`, using the [documented Cursor plugin format](https://cursor.com/docs/reference/plugins). It points to the same skill directory and `.mcp.json`; there is no additional runtime.

Grok Bot is the persistent cloud agent product. Grok Build is the separate terminal coding agent. [Grok Bot supports Cursor plugins](https://x.ai/bot/guides/grok-bot-101), but compatibility of this specific package remains unverified.

This plugin is **not published in Grok Bot's Marketplace**. The [official native connection flow](https://docs.x.ai/grok-bot/computer-and-apps) is Marketplace → choose a plugin → Add → authenticate → attach it with `@`. The GitHub repository and Grok Build commands do not install a native Grok Bot connector.

Before claiming native support, verify Marketplace installation, TrialStack OAuth, and a read-only `list_organizations` request. Then select a returned organization and read its trials. Do not execute clinical changes as an installation smoke test.

For package testing before publication, Cursor documents [local plugin loading](https://cursor.com/docs/plugins). Copy the complete nested plugin directory into its local plugin directory; do not copy only the manifest. A passing Cursor load validates packaging only, not Grok Bot authentication or runtime compatibility.

## Install in Grok Build

From a local checkout:

```bash
grok plugin validate /path/to/trialstack-catalyst/plugins/trialstack-catalyst
grok plugin install /path/to/trialstack-catalyst/plugins/trialstack-catalyst --trust
grok plugin enable trialstack-catalyst
```

Or add this repository as a marketplace and install by name:

```bash
grok plugin marketplace add /path/to/trialstack-catalyst
grok plugin install trialstack-catalyst --trust
```

From GitHub:

```bash
grok plugin marketplace add trialstack/trialstack-catalyst
grok plugin install trialstack-catalyst --trust
grok plugin enable trialstack-catalyst
```

Start a new session after installation. Call `list_organizations` first and
use the chosen organization for later calls.

## Install in Codex

From a local checkout:

```bash
codex plugin marketplace add /path/to/trialstack-catalyst
codex plugin add trialstack-catalyst@trialstack
```

From GitHub:

```bash
codex plugin marketplace add trialstack/trialstack-catalyst --ref main
codex plugin add trialstack-catalyst@trialstack
```

Start a new Codex task after installation and complete the TrialStack OAuth
flow. Use `list_organizations` before organization-scoped work and pass
`organizationId` explicitly when the requested organization is not the OAuth
default.

## Provenance

The plugin files (`plugins/…`, `.agents/plugins/marketplace.json`) remain
canonical in the TrialStack monorepo and are synced here byte-identical with
its `export-dist.sh` allowlisted export procedure. This repository owns only
its root `README.md`, `LICENSE`, and the root Grok and Cursor marketplace indexes.

## License

MIT. See [LICENSE](LICENSE).
