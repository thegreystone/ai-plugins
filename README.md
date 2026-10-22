# AI Plugins

Plugin marketplace for Claude, ChatGPT and Codex.

## Plugins

| Plugin | What it does |
| --- | --- |
| [OpenJDK Helper](https://github.com/thegreystone/openjdk-helper) | Read-only tools for OpenJDK contributors and project leads: census roles, pull requests, issues and backports, JEPs, release schedules and project processes. |
| [Mnemic](https://github.com/thegreystone/mnemic) | Persistent local memory shared by all your AI assistants: what you told them, kept as dated facts in one SQLite file on your machine and recalled before they answer. Available for Claude Code and Codex. |

OpenJDK Helper connects to the free hosted MCP service at `https://api.hirt.se/openjdk/mcp`. It requires no service account, local Java runtime or native executable. See its [privacy notice](https://github.com/thegreystone/openjdk-helper/blob/main/PRIVACY.md) and [service terms](https://github.com/thegreystone/openjdk-helper/blob/main/TERMS.md).

Try:

- "Who are the Reviewers in the JMC project?"
- "Where has JDK-8394772 been fixed, and which backports are still open?"
- "Summarize the process for nominating a new OpenJDK Committer."

Mnemic runs on your machine, and everything it remembers stays in one SQLite file in a folder you choose. The Claude Code plugin downloads a universal MCP Bundle from [Mnemic's releases](https://github.com/thegreystone/mnemic/releases), containing macOS Apple silicon and Windows x64 binaries. The Codex plugin uses a Node.js 18+ launcher to download and cache the pinned standalone binary for macOS Apple silicon, Windows x64, or Linux x64/ARM64. Neither requires Java or an npm package. Claude Code users on Linux [install the Linux binary by hand](https://github.com/thegreystone/mnemic#install). Claude Desktop users install the per-platform `.mcpb` from the release page because Desktop does not accept bundles referenced by URL.

Try:

- "Remember that we decided to use SQLite for the project."
- "Where did I work in 2015?"
- "What is Mnemic's status?"

## Install (Claude)

Add the marketplace in Claude Code:

```
/plugin marketplace add thegreystone/ai-plugins
```

Then install the plugins you want:

```
/plugin install openjdk-helper@hirt-ai-plugins
/plugin install mnemic@hirt-ai-plugins
```

Or from the shell:

```sh
claude plugin marketplace add thegreystone/ai-plugins
claude plugin install openjdk-helper@hirt-ai-plugins
claude plugin install mnemic@hirt-ai-plugins
```

Mnemic needs your name before its server starts. In a session, open `/plugin`, choose **mnemic** on the **Installed** tab, choose **Configure**, enter the name, and start a new session. This is the only way to set it: the shell `--config` flag and `claude plugin configure` do not reach a bundle fetched from a URL.

In Claude Desktop, open *Settings → Extensions → Browse → Add marketplace* and paste `https://github.com/thegreystone/ai-plugins`.

To pick up plugin changes:

```sh
claude plugin marketplace update hirt-ai-plugins
claude plugin update openjdk-helper@hirt-ai-plugins
claude plugin update mnemic@hirt-ai-plugins
```

## Install (Codex)

Add the marketplace:

```sh
codex plugin marketplace add thegreystone/ai-plugins
```

Open the Plugins Directory in the ChatGPT desktop app, select **Marcus Hirt's AI Plugins**, and install **OpenJDK Helper** and/or **Mnemic**. Enable the installed plugin in a new conversation.

Mnemic requires Node.js 18+ and your name before its server can start. Invoke its setup skill, supply your name and optionally a data home, and let it configure the launcher and download the binary. Start a new conversation, then ask for Mnemic's status. Use the same data home as another Mnemic client to share memories. See the [Codex plugin guide](https://github.com/thegreystone/mnemic/blob/main/plugin-codex/README.md) for configuration, supported platforms and manual binary overrides.

To refresh the marketplace:

```sh
codex plugin marketplace upgrade hirt-ai-plugins
```

Client and workspace settings determine which plugin installation options are available. See the [official installation documentation](https://developers.openai.com/plugins/build/plugins#add-a-marketplace-from-the-cli).

## Documentation

See [maintaining the marketplace](docs/maintaining.md) for its structure, plugin sources, local testing and updates.

The marketplace metadata and documentation use the [BSD 3-Clause license](LICENSE). Each plugin has its own license and service policies.
