# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted LearnTheDao MCP server. The four captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `list_chapters`, `get_chapter`, `search_chapters`, `consult` — the full captured 4-tool surface, no omissions.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `taochapters error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `LEARNTHEDAO_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
