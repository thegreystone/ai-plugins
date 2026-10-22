# AI Plugins

Plugin marketplace for Claude, ChatGPT and Codex.

## Plugins

| Plugin | What it does |
| --- | --- |
| [OpenJDK Helper](https://github.com/thegreystone/openjdk-helper) | Read-only tools for OpenJDK contributors and project leads: census roles, pull requests, issues and backports, JEPs, release schedules and project processes. |

OpenJDK Helper connects to the free hosted MCP service at `https://api.hirt.se/openjdk/mcp`. It requires no service account, local Java runtime or native executable. See its [privacy notice](https://github.com/thegreystone/openjdk-helper/blob/main/PRIVACY.md) and [service terms](https://github.com/thegreystone/openjdk-helper/blob/main/TERMS.md).

Try:

- "Who are the Reviewers in the JMC project?"
- "Where has JDK-8394772 been fixed, and which backports are still open?"
- "Summarize the process for nominating a new OpenJDK Committer."

## Install (Claude)

Add the marketplace in Claude Code:

```
/plugin marketplace add thegreystone/ai-plugins
```

Then install the plugin:

```
/plugin install openjdk-helper@hirt-ai-plugins
```

Or from the shell:

```sh
claude plugin marketplace add thegreystone/ai-plugins
claude plugin install openjdk-helper@hirt-ai-plugins
```

In Claude Desktop, open *Settings → Extensions → Browse → Add marketplace* and paste `https://github.com/thegreystone/ai-plugins`.

To pick up plugin changes:

```sh
claude plugin marketplace update hirt-ai-plugins
claude plugin update openjdk-helper@hirt-ai-plugins
```

## Install (Codex)

Add the marketplace:

```sh
codex plugin marketplace add thegreystone/ai-plugins
```

Open the Plugins Directory in the ChatGPT desktop app, select **Marcus Hirt's AI Plugins**, and install **OpenJDK Helper**. Enable it in a new conversation.

To refresh the marketplace:

```sh
codex plugin marketplace upgrade hirt-ai-plugins
```

Client and workspace settings determine which plugin installation options are available. See the [official installation documentation](https://developers.openai.com/plugins/build/plugins#add-a-marketplace-from-the-cli).

## Documentation

See [maintaining the marketplace](docs/maintaining.md) for its structure, plugin sources, local testing and updates.

The marketplace metadata and documentation use the [BSD 3-Clause license](LICENSE). Each plugin has its own license and service policies.
