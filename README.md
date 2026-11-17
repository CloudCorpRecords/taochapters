# taochapters

Read the Tao Te Ching through your own machine's tool list. taochapters is a local forwarding client for the hosted LearnTheDao MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every read to LearnTheDao, returning the chapters exactly as the service sends them back.

The surface covers all 81 chapters in two renderings. You can list every chapter with its title and the attribution blocks for each rendering, pull one chapter in both renderings at once, search the full corpus for a half-remembered line, or describe a situation in your own words and be matched to the chapter that speaks to it. Nothing is cached, stored or computed locally — the package is a pipe, not a copy of the text.

## The tools

- **`list_chapters`** — Every chapter title in order, plus the attribution block for each available rendering. Takes no input; a good first call for orientation.
- **`get_chapter`** — One chapter, numbered 1 to 81, in both renderings: the public-domain James Legge (1891) translation and Dao De Kooi, an original rendering by David Kooi, each with its own attribution and citation string. `number` is required.
- **`search_chapters`** — Search the whole corpus for a word or phrase and get the matching lines with their chapter numbers. `query` is required (at least 2 characters); `limit` optionally caps the matches at 50.
- **`consult`** — Describe what someone is actually facing and get the chapter that speaks to it, plus the one line to hold. This matches the situation rather than the words. `situation` is required (at least 3 characters); `limit` optionally caps the matches at 5.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/taochapters mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "taochapters": {
      "command": "npx",
      "args": ["-y", "@reneturcios/taochapters", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/taochapters
docker run --rm ghcr.io/cloudcorprecords/taochapters get_chapter '{"number":63}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
taochapters list_chapters '{}'
taochapters get_chapter '{"number":63}'
taochapters search_chapters '{"query":"without doing"}'
taochapters consult '{"situation":"a coworker took credit for my work"}'
taochapters --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `LEARNTHEDAO_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://learnthedao.com/api/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `taochapters error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface, exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
