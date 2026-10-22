# Maintaining the marketplace

This repository is one marketplace with two catalogs, one per client family. Both share the stable identifier `hirt-ai-plugins` and list the same plugins.

| Client | Catalog | Display name |
| --- | --- | --- |
| Claude Code, Claude Desktop | [.claude-plugin/marketplace.json](../.claude-plugin/marketplace.json) | `hirt-ai-plugins` |
| ChatGPT, Codex | [.agents/plugins/marketplace.json](../.agents/plugins/marketplace.json) | **Marcus Hirt's AI Plugins** |

The two formats differ, so keep the catalogs separate and change both when adding or updating a plugin. Codex also reads the Claude catalog location as a legacy fallback; the dedicated OpenAI catalog takes precedence.

## Plugin sources

OpenJDK Helper is fetched from its own repository in both catalogs:

- Repository: `https://github.com/thegreystone/openjdk-helper.git`
- Plugin directory: `plugin`
- Ref: `main`

The client retrieves the plugin from that repository. OpenJDK Helper's skill and assets are maintained in its [plugin source](https://github.com/thegreystone/openjdk-helper/tree/main/plugin). The marketplace contains its catalog entries.

### Claude catalog

The Claude entry uses a `git-subdir` source, since Claude's `github` source type cannot select a subdirectory. The plugin directory carries its own Claude manifest (`.claude-plugin/plugin.json`) and MCP configuration (`.mcp.json`), so the entry only adds what the catalog listing needs: `description`, `category` and `tags`. Author, license, homepage, icon and policy URLs come from the plugin manifest.

Neither the entry nor the plugin manifest sets a `version`. Claude Code then versions a `git-subdir` plugin by its commit SHA, so every push to the plugin's `main` branch reaches users on `claude plugin update` without a release step. Do not add a `version` to the entry: it would freeze users on the cached copy until the string changes.

### OpenAI catalog

The OpenAI entry uses the equivalent `git-subdir` source. The plugin directory's `plugin.json`, `mcp.json` and skill are in OpenAI's portable Agent Plugin format. The entry makes installation available and uses the on-install authentication policy. The hosted OpenJDK Helper MCP connection itself requires no authentication.

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

This tests the catalog entry, not unpushed plugin changes: the `git-subdir` source fetches `plugin/` from GitHub even when the marketplace is local. To test plugin changes before they are pushed, use the relative-path development marketplace described in the [OpenJDK Helper plugin guide](https://github.com/thegreystone/openjdk-helper/blob/main/docs/plugin.md#claude-code).

Catalog edits in a local-path marketplace take effect at the next session start or after `/reload-plugins`. Remove the local registration with `claude plugin marketplace remove hirt-ai-plugins` before adding the GitHub one; the two cannot share a name, and removal uninstalls the plugin installed from it.

### Codex

```powershell
codex plugin marketplace add "C:\Users\Marcus\git\me\ai-plugins"
```

Then install OpenJDK Helper from the Plugins Directory in the ChatGPT desktop app and enable it in a new conversation. Reinstall after catalog edits: local installs use a cached copy.

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

A Git-backed marketplace supports local and team distribution. Publishing to Anthropic's plugin directory or OpenAI's universal public directory is a separate submission and review process for each.

See the [Claude marketplace reference](https://code.claude.com/docs/en/plugins/marketplace-reference), the [OpenAI marketplace reference](https://developers.openai.com/plugins/build/plugins#marketplace-metadata) and the [OpenJDK Helper plugin guide](https://github.com/thegreystone/openjdk-helper/blob/main/docs/plugin.md).
