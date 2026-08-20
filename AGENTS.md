# AGENTS.md

## Cursor Cloud specific instructions

This repository is a single product: a **Google Forms MCP server** (TypeScript, Node.js). It is a
Model Context Protocol server that speaks **JSON-RPC over stdio** — it is NOT an HTTP/web service, so
there is no port to open or page to load. You interact with it through an MCP client attached to its
stdin/stdout. Standard build/run commands live in `README.md` and `package.json` `scripts`.

### Build / run
- Dependencies: `npm install` (already run by the environment update script).
- Build before running: `npm run build` (runs `tsc` → outputs to `build/`, which is gitignored). There
  is no watch/hot-reload — re-run `npm run build` after editing anything in `src/`.
- Run the server: `npm run start` (i.e. `node build/index.js`).

### Credentials (important, non-obvious)
- The server reads `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, and `GOOGLE_REFRESH_TOKEN` at startup and
  **throws immediately** if any are missing. That error is the expected guard, not a build/setup failure.
- Real Google Forms actions (e.g. `create_form`) make live calls to Google's OAuth token endpoint and the
  Forms API. Network egress to Google works in this environment; the only thing needed for a real form is
  valid credentials. With dummy credentials, calls reach Google and return `invalid_client` — which still
  proves the full code path works.
- `npm run get-refresh-token` starts a local browser-based OAuth flow on `http://localhost:3000` and calls
  `open` to launch a browser. This is not usable headless in the cloud VM; obtain the refresh token on a
  machine with a browser and supply it as a secret.

### Smoke-testing without real credentials
Set dummy `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` / `GOOGLE_REFRESH_TOKEN` so the startup guard passes,
then connect an MCP stdio client (`@modelcontextprotocol/sdk` `Client` + `StdioClientTransport`) to
`node build/index.js`, run the MCP handshake, and call `tools/list`. It returns the 5 tools
(`create_form`, `add_text_question`, `add_multiple_choice_question`, `get_form`, `get_form_responses`).

### Tests / lint
There are no automated tests and no linter/formatter configured in this repo.
