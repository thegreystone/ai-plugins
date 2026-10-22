# Maintaining the marketplace

This repository is one marketplace with two catalogs, one per client family. Both share the stable identifier `hirt-ai-plugins`. OpenJDK Helper and Mnemic are listed in both catalogs. Mnemic uses a separate plugin directory for each client family.

| Client | Catalog | Display name |
| --- | --- | --- |
| Claude Code, Claude Desktop | [.claude-plugin/marketplace.json](../.claude-plugin/marketplace.json) | `hirt-ai-plugins` |
| ChatGPT, Codex | [.agents/plugins/marketplace.json](../.agents/plugins/marketplace.json) | **Marcus Hirt's AI Plugins** |

The two formats differ, so keep the catalogs separate and change both when adding or updating a plugin that both list. Codex also reads the Claude catalog location as a legacy fallback; the dedicated OpenAI catalog takes precedence.

## Plugin sources

### OpenJDK Helper

OpenJDK Helper is fetched from its own repository in both catalogs:

- Repository: `https://github.com/thegreystone/openjdk-helper.git`
- Plugin directory: `plugin`
- Ref: `main`

The client retrieves the plugin from that repository. OpenJDK Helper's skill and assets are maintained in its [plugin source](https://github.com/thegreystone/openjdk-helper/tree/main/plugin). The marketplace contains its catalog entries.

#### Claude catalog

The Claude entry uses a `git-subdir` source, since Claude's `github` source type cannot select a subdirectory. The plugin directory carries its own Claude manifest (`.claude-plugin/plugin.json`) and MCP configuration (`.mcp.json`), so the entry only adds what the catalog listing needs: `description`, `category` and `tags`. Author, license, homepage, icon and policy URLs come from the plugin manifest.

Neither the entry nor the plugin manifest sets a `version`. Claude Code then versions a `git-subdir` plugin by its commit SHA, so every push to the plugin's `main` branch reaches users on `claude plugin update` without a release step. Do not add a `version` to the entry: it would freeze users on the cached copy until the string changes.

#### OpenAI catalog

The OpenAI entry uses the equivalent `git-subdir` source. The plugin directory's `plugin.json`, `mcp.json` and skill are in OpenAI's portable Agent Plugin format. The entry makes installation available and uses the on-install authentication policy. The hosted OpenJDK Helper MCP connection itself requires no authentication.

### Mnemic

Mnemic runs locally. Its native binaries are published on GitHub Releases rather than committed to git or published to npm. Both catalogs use a `git-subdir` source:

- Repository: `https://github.com/thegreystone/mnemic.git`
- Ref: `main`

The Claude catalog selects `plugin`; the OpenAI catalog selects `plugin-codex`.

#### Claude catalog

That directory holds only the manifest, the `mnemic` skill and a README. The manifest's `mcpServers` field is the URL of the universal MCP Bundle attached to a Mnemic release, for example `https://github.com/thegreystone/mnemic/releases/download/v0.3.2/mnemic-0.3.2-universal.mcpb`. Installing the plugin fetches only the plugin directory; at the next session start Claude Code downloads the bundle into `.mcpb-cache/` under the installed plugin, extracts it, and reads the server configuration from the bundle's own manifest. The bundle's `user_config` is the plugin's configuration, reachable only through `/plugin` → **Installed** → **mnemic** → **Configure**: `claude plugin install --config` and `claude plugin configure` report that a URL-fetched bundle cannot be configured from the shell, and `claude plugin details` lists no MCP server before the first session. The server does not start until the owner name is set, and from Claude Code 2.1.285 on the install message and `/plugin` say so; before that the server was skipped silently (anthropics/claude-code#95683). Values are stored under `pluginConfigs["mnemic@hirt-ai-plugins"].mcpServers.mnemic` in the user's `settings.json`.

Claude Desktop rejects a plugin whose bundle is referenced by URL (anthropics/claude-code#96144), so the Mnemic entry is usable from Claude Code only; Desktop users take the per-platform `.mcpb` from the release page.

The universal bundle holds the macOS Apple silicon and Windows x64 binaries, with the bundle manifest's `platform_overrides` choosing one at launch. Linux is left out on purpose: the bundle would double in size, and Linux users install the standalone binary from the release page. Keep [pack-universal.sh](https://github.com/thegreystone/mnemic/blob/main/mcpb/pack-universal.sh) to those two platforms.

Updating Mnemic means changing the release version in that one URL in the plugin manifest after the release's universal bundle is final. The Windows binary is signed after the release workflow runs, and the signing script re-packs the universal bundle, so point at a release only once that has happened. Because the bundle URL pins the version, a manifest `version` field is not needed; the `git-subdir` source tracks commits as for OpenJDK Helper.

#### OpenAI catalog

The `plugin-codex` directory contains a portable Agent Plugin manifest, `mcp.json`, skills, and a Node.js 18+ launcher. The MCP configuration starts the launcher over stdio. During setup, the launcher downloads the pinned standalone native executable from GitHub Releases into a local cache. The supported platforms are Linux x64/ARM64, macOS Apple silicon, and Windows x64. Intel macOS is not supported. It does not parse MCP Bundles or require an npm publication.

The setup skill runs the launcher with `--configure --owner NAME`, optionally adding `--home PATH` and `--language LANGUAGE`, then uses `--install` to prefetch the binary. Restart the conversation after setup and request Mnemic's status. Owner and data-home settings are saved outside the installed plugin so plugin updates retain them. `MNEMIC_CODEX_CONFIG` and `MNEMIC_CODEX_CACHE` override the configuration file and binary cache locations; explicit `MNEMIC_*` server settings override saved configuration, and `MNEMIC_BINARY` selects an existing executable instead of downloading one. See the [launcher guide](https://github.com/thegreystone/mnemic/blob/main/plugin-codex/README.md) for the exact paths and supported options.

To update the Codex binary, change the launcher's pinned release version and the version in `plugin-codex/plugin.json` together after the matching standalone release assets are finalized. Validate the asset names and supported platforms before publishing. This is independent of the Claude plugin's MCP Bundle URL. Keep `plugin` and `plugin-codex` client configuration separate while preserving their shared `mnemic` plugin name.

## Local testing

### Claude

Validate the catalog, then register this checkout as a marketplace and install from it:

```sh
claude plugin validate .
claude plugin marketplace add "C:\Users\Marcus\git\me\ai-plugins"
claude plugin install openjdk-helper@hirt-ai-plugins
claude plugin details openjdk-helper@hirt-ai-plugins
claude mcp list
```

`details` should list one skill and one MCP server, and `mcp list` should show `plugin:openjdk-helper:openjdk-helper` as connected. Start a new session and try a prompt such as "Who are the Reviewers in the JMC project?"; the answer should come from the OpenJDK Helper tools with census links.

This tests the catalog entry, not unpushed plugin changes: the `git-subdir` source fetches `plugin/` from GitHub even when the marketplace is local. To test plugin changes before they are pushed, use the relative-path development marketplace described in the [OpenJDK Helper plugin guide](https://github.com/thegreystone/openjdk-helper/blob/main/docs/plugin.md#claude-code), or load a plugin directory in place with `claude --plugin-dir <path>`.

For Mnemic, `claude plugin validate <mnemic>/plugin` checks the manifest; the bundle download and configuration are exercised only by a real session. After installing, start a session, open `/plugin` → **Installed** → **mnemic** → **Configure**, enter a name and a test data home, start a new session and ask for Mnemic's status. The server appears in `/mcp` as `plugin:mnemic:mnemic` and its tools as `mcp__plugin_mnemic_mnemic__<tool>`. Use a separate data home if the machine already has a Mnemic memory you do not want the test to touch, and remove any user-level `claude mcp add mnemic` entry once the plugin works, or both servers will run.

Catalog edits in a local-path marketplace take effect at the next session start or after `/reload-plugins`. Remove the local registration with `claude plugin marketplace remove hirt-ai-plugins` before adding the GitHub one; the two cannot share a name, and removal uninstalls the plugin installed from it.

### Codex

```powershell
codex plugin marketplace add "C:\Users\Marcus\git\me\ai-plugins"
```

Then install OpenJDK Helper and/or Mnemic from the Plugins Directory in the ChatGPT desktop app and enable the plugin in a new conversation. Reinstall after catalog edits: local installs use a cached copy. The Mnemic entry still fetches `plugin-codex` from GitHub, so local marketplace registration does not test unpushed Mnemic plugin changes.

For Mnemic, first parse `plugin-codex/plugin.json` and `plugin-codex/mcp.json`, run the launcher syntax check and tests described in its [guide](https://github.com/thegreystone/mnemic/blob/main/plugin-codex/README.md), and check the pinned release's standalone assets. A real client test remains necessary: install the plugin, run its setup skill with a test owner and a separate data home, restart the conversation, and ask for Mnemic's status. Confirm the reported owner and data home, and confirm the tools come from the plugin. Avoid using an existing personal memory for a smoke test or running a second manually configured Mnemic server alongside the plugin.

## Adding or updating entries

Add an entry to each catalog's `plugins` array. In both, the entry name must match the plugin's own name. Use `ref` to select a branch or tag, or `sha` to select a specific commit. Review source changes before changing a pinned revision. OpenJDK Helper currently follows `main` in both catalogs.

### Claude

A Claude entry needs `name`, a supported source and a `description`; `category` and `tags` help users find it. Leave `version` out of the entry when the plugin follows a branch, so Claude Code tracks commits. If the plugin directory has no `.claude-plugin/plugin.json`, the entry must also carry the manifest fields, including `mcpServers`; if it has one, those fields belong in the plugin and the entry's `mcpServers` would be ignored.

Before publishing a Claude catalog change:

1. Run `claude plugin validate .` and check the source repository, ref and plugin path.
2. Register this checkout and install the plugin as described under [Local testing](#claude).
3. Test a representative request in a new session.
4. Commit and push, then remove the local marketplace, add `thegreystone/ai-plugins` and install again from GitHub.

### Codex

A Codex entry needs `name`, a supported source, installation and authentication policies, and a category.

Before publishing a Codex catalog change:

1. Parse the JSON and check the source repository, ref and plugin path.
2. Register this checkout as described under [Local testing](#codex).
3. Install the plugin from the desktop Plugins Directory and test a representative request in a new conversation.
4. Commit and push, then check installation through the GitHub marketplace URL.

For Mnemic, publish the new or updated `plugin-codex` directory in the Mnemic repository and confirm that its pinned native release assets are available before publishing the marketplace entry. Commit and push the OpenAI catalog and documentation in this repository as well. A JSON or launcher check does not establish that marketplace installation and a new client session work; record that smoke test separately.

A Git-backed marketplace supports local and team distribution. Publishing to Anthropic's plugin directory or OpenAI's universal public directory is a separate submission and review process for each.

See the [Claude marketplace reference](https://code.claude.com/docs/en/plugins/marketplace-reference), the [OpenAI marketplace reference](https://developers.openai.com/plugins/build/plugins#marketplace-metadata) and the [OpenJDK Helper plugin guide](https://github.com/thegreystone/openjdk-helper/blob/main/docs/plugin.md).
