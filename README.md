# AI Plugins

Plugin marketplace for Claude, ChatGPT and Codex.

## Plugins

| Plugin | What it does |
| --- | --- |
| [OpenJDK Helper](https://github.com/thegreystone/openjdk-helper) | Read-only tools for OpenJDK contributors and project leads: census roles, pull requests, issues and backports, JEPs, release schedules and project processes. |
| [Mnemic](https://github.com/thegreystone/mnemic) | Persistent local memory shared by all your AI assistants: what you told them, kept as dated facts in one SQLite file on your machine and recalled before they answer. Available for Claude Code and Codex. |
| [Email MCP](https://github.com/thegreystone/email-mcp) | Read, search, triage and draft email over IMAP and SMTP. Runs locally, with sending and permanent deletion off by default. Available for Claude Code and Codex. |

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

Email MCP connects to your own mail accounts over IMAP and SMTP. The Claude Code plugin uses the released universal MCP Bundle for macOS Apple silicon and Windows x64; the Codex plugin uses a Node.js 18+ launcher for those platforms and Linux x64/ARM64. Enter credentials privately in Claude's plugin configuration or the Codex launcher's local settings file. See the [plugin guide](https://github.com/thegreystone/email-mcp/blob/master/docs/PLUGINS.md).

Try:

- "List my configured email accounts."
- "Summarize my unread messages from today."
- "Save a draft reply for me to review."

## Install (Claude)

Add the marketplace in Claude Code:

```
/plugin marketplace add thegreystone/ai-plugins
```

Then install the plugins you want:

```
/plugin install openjdk-helper@hirt-ai-plugins
/plugin install mnemic@hirt-ai-plugins
/plugin install email-mcp@hirt-ai-plugins
```

Or from the shell:

```sh
claude plugin marketplace add thegreystone/ai-plugins
claude plugin install openjdk-helper@hirt-ai-plugins
claude plugin install mnemic@hirt-ai-plugins
claude plugin install email-mcp@hirt-ai-plugins
```

Mnemic needs your name before its server starts. In a session, open `/plugin`, choose **mnemic** on the **Installed** tab, choose **Configure**, enter the name, and start a new session. This is the only way to set it: the shell `--config` flag and `claude plugin configure` do not reach a bundle fetched from a URL.

Email MCP users open `/plugin` → **Installed** → **email-mcp** → **Configure**, enter the IMAP and SMTP settings, and start a new session. Keep passwords out of chat. Like Mnemic, Email MCP's URL-backed bundle is for Claude Code; Claude Desktop users install the per-platform `.mcpb` from the [release page](https://github.com/thegreystone/email-mcp/releases/latest).

In Claude Desktop, open *Settings → Extensions → Browse → Add marketplace* and paste `https://github.com/thegreystone/ai-plugins`.

To pick up plugin changes:

```sh
claude plugin marketplace update hirt-ai-plugins
claude plugin update openjdk-helper@hirt-ai-plugins
claude plugin update mnemic@hirt-ai-plugins
claude plugin update email-mcp@hirt-ai-plugins
```

## Install (Codex)

Add the marketplace:

```sh
codex plugin marketplace add thegreystone/ai-plugins
```

Open the Plugins Directory in the ChatGPT desktop app, select **Marcus Hirt's AI Plugins**, and install **OpenJDK Helper**, **Mnemic** and/or **Email MCP**. Enable the installed plugin in a new conversation.

Mnemic requires Node.js 18+ and your name before its server can start. Invoke its setup skill, supply your name and optionally a data home, and let it configure the launcher and download the binary. Start a new conversation, then ask for Mnemic's status. Use the same data home as another Mnemic client to share memories. See the [Codex plugin guide](https://github.com/thegreystone/mnemic/blob/main/plugin-codex/README.md) for configuration, supported platforms and manual binary overrides.

For Email MCP, run its **Setup** workflow to create a local configuration template, enter mailbox credentials privately in an editor, and prefetch the native executable. Start a new conversation and ask it to list configured accounts. Its [Codex plugin guide](https://github.com/thegreystone/email-mcp/blob/master/plugin-codex/README.md) covers configuration, environment overrides and supported platforms.

To refresh the marketplace:

```sh
codex plugin marketplace upgrade hirt-ai-plugins
```

Client and workspace settings determine which plugin installation options are available. See the [official installation documentation](https://developers.openai.com/plugins/build/plugins#add-a-marketplace-from-the-cli).

## Documentation

See [maintaining the marketplace](docs/maintaining.md) for its structure, plugin sources, local testing and updates.

The marketplace metadata and documentation use the [BSD 3-Clause license](LICENSE). Each plugin has its own license and service policies.
