# BUILD REPORT — taochapters

Clean-room rebuild of `com.learnthedao/learn-the-dao` v1.0.0
(remote-only MCP server at `https://learnthedao.com/api/mcp`), built
2026-10-28 as a forwarding client. Package: `@reneturcios/taochapters`
0.1.0, registry name `io.github.CloudCorpRecords/taochapters`.

## Surface built

**Full surface: 4 of 4 captured tools built, no omissions.**

| Tool | Required fields as served |
|------|---------------------------|
| `list_chapters` | NO `required` key in the capture — served verbatim, none invented |
| `get_chapter` | `number` |
| `search_chapters` | `query` |
| `consult` | `situation` |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured (no `title` key exists in the
capture, so none is served; every `inputSchema` carries
`additionalProperties: false`, matching the fixture). The tool
literals were generated programmatically from
`harness/fixtures/learnthedao-tools-original.json` by the script at
`/tmp/laneb4-taochapters-scratch/gen.py` (parse + re-serialise; the
emitted fragment was placed into `src/forward.ts`) — never retyped —
and a post-generation check (the registration-integrity test suite,
plus the parity harness) confirmed the served literals deep-equal
the fixture's 4 tool objects, in order. Tools are registered via
the MCP SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tool names,
descriptions, required fields (none for `list_chapters`; `number`
for `get_chapter`; `query` for `search_chapters`; `situation` for
`consult`) and property shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://learnthedao.com/api/mcp`.
- Env override `LEARNTHEDAO_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`LEARNTHEDAO_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and that
  no `X-API-Key` header is ever sent; upstream requests carry only
  `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `taochapters error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests, and no live call is
  claimed beyond the coordinator's tools/list capture.

## Lockfile

- Route: CLEAN Route 3 from birth — generated in the build dir with
  no `node_modules` present via
  `npm install --package-lock-only --ignore-scripts --offline`.
- Verified: parses as JSON, `name == @reneturcios/taochapters`,
  exactly **128** package entries.
- sha256: `2a3a6dd8f4fd32ac341576ae4bbb7089b84f9d8954f59be077d337c7c71a863f`
- `npm ci --ignore-scripts --offline` then populated `node_modules`
  cleanly (0 vulnerabilities reported by npm).

## server.json

- Description: "Tao Te Ching chapters in two renderings: list, read, search, and match by situation." — **84 chars** (≤ 100).
- npm entry carries `version: 0.1.0`; OCI entry
  `ghcr.io/cloudcorprecords/taochapters:0.1.0` carries **no**
  `version` field (verified programmatically).
- Dockerfile label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/taochapters"`
  present; base image `node:20-alpine` in both stages;
  `ENTRYPOINT ["node", "/app/dist/cli.js"]`, `CMD ["mcp"]`.
- `publish-container.yml` IMAGE is
  `ghcr.io/cloudcorprecords/taochapters` and its smoke test asserts
  the server name `taochapters`.

## Verification (own run, 2026-10-28)

- `npx tsc --noEmit` — exit 0.
- `npm run build` — exit 0 (`dist/cli.js` produced; `node dist/cli.js --version` prints `0.1.0`).
- `npm test` — **53 tests, 53 pass, 0 fail** (8 suites: registration
  integrity incl. served tools/list deep-equal to the fixture's 4
  tools in order; session establishment and reuse; no-Authorization
  assertions under several environments; env override; verbatim
  argument forwarding; result pass-through incl. upstream isError;
  SSE parsing; error rendering; MCP layer validation; CLI behaviour
  incl. `--version` printing `0.1.0`). Fake-fetch only — no live
  network calls.
- Parity: `python3 harness/parity.py --original harness/fixtures/learnthedao-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/taochapters/dist/cli.js mcp" --out builds/taochapters/parity-out`
  → **verdict PASS, parity 100.0** (4/4 identical, no missing, no
  extra, no schema diffs). Report in `parity-out/parity.json`.
- Full 19-file factory set present (package.json, server.json,
  Dockerfile, tsconfig.json, .gitignore, package-lock.json,
  README.md, LICENSE, CHANGELOG.md, src/{index,cli,mcp,types,forward}.ts,
  test/forward.test.ts, .github/workflows/{ci,publish,publish-container,registry-publish}.yml).
