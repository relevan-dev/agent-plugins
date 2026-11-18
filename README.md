# Relevan agent plugin

Relevan is search that learns from feedback. This plugin adds the hosted Relevan MCP server and the Relevan skills to your coding agent.

The MCP server is at `https://api.relevan.dev/v1/mcp`. Its tools create an index, load documents, search, and send feedback.

The skills:

- `relevan-setup`: Set up Relevan search from the start: authenticate, create an index, load documents, run the first search, and send the first feedback. Use when a user asks you to add Relevan search to a product.
- `relevan-ranking-signals`: Find the data that tells Relevan which result is the best one, such as prices, sales, or activity, and add it to each document. Use before you design a Relevan mapping.
- `relevan-mapping`: Draft and deploy a RelevanMapping from a sample of documents. Use when a Relevan index has no mapping, or when the documents get new fields.
- `relevan-feedback`: Send click, conversion, and llm_judgment feedback to Relevan after a search, so the ranking gets better. Use when you build search into an app, or when an agent acts on a search result.

<!-- Setup video: replace this comment with the link to the 3-minute setup video. -->

## Install

Claude Code:

```sh
claude plugin marketplace add relevan-dev/agent-plugins
claude plugin install relevan@relevan
```

GitHub Copilot CLI:

```sh
copilot plugin marketplace add relevan-dev/agent-plugins
copilot plugin install relevan@relevan
```

Codex:

```sh
codex plugin marketplace add relevan-dev/agent-plugins
codex plugin add relevan@relevan
```

Start a new session after the install.

## Sign in

The MCP server uses OAuth, so you need no API key.

1. Connect the `relevan` MCP server:
   - Claude Code: run `/mcp`, select `relevan`, then select Authenticate.
   - Copilot CLI: run `/mcp`, then select `relevan`.
   - Codex: run `codex mcp login relevan`.
2. Your browser opens the Relevan console. Sign in, or create a free account.
3. Pick an organization, then approve the access:
   - `read` lets the agent search and send feedback.
   - `write` also lets the agent create indexes and load documents. Only an organization owner can grant it.

To disconnect the agent, open the Security page of the console.

## Try it

Ask the agent:

> Use the relevan-setup skill. Create an index for my product catalog, load a sample of it, and search it.

## Update

The plugin has no version number. An update brings the newest commit of this repository.

- Claude Code: `claude plugin marketplace update relevan`, then `claude plugin update relevan@relevan`
- Copilot CLI: `copilot plugin marketplace update relevan`, then `copilot plugin update relevan`
- Codex: `codex plugin marketplace upgrade relevan`

## Learn more

- Docs: https://docs.relevan.dev
- MCP server guide: https://docs.relevan.dev/guides/mcp-server

## License

Apache-2.0. See [LICENSE](LICENSE).
