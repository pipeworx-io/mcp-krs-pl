# @pipeworx/krs-pl

Poland's National Court Register (KRS): look up a Polish company, foundation or association by KRS number and get its legal name, legal form, REGON and NIP, registration date, registered address, share capital, main activity (PKD) and whether it has liquidation, bankruptcy or dissolution entries.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2018+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `krs_company_get(krs, register?)` — one entity by KRS number (1-10 digits; padded to 10). Tries the entrepreneurs register, then the associations/foundations/public-bodies register. An unknown number returns `found:false` with `reason`.

Every response carries `data_as_of` (the date the extract is current to, as stated by the register) and `source`.

## Auth

Keyless.

## Data sources

- Ministry of Justice of Poland, National Court Register (KRS), current extract.

## Notes

- The register can only be looked up by KRS number. It offers no name search, so this pack cannot turn a company name into a number. Take the KRS number from a document, a website footer, or the NIP/REGON pack.
- Officers, liquidators, trustees and shareholders are not returned. The register masks their names and personal numbers, and this pack drops those sections entirely rather than pass masked personal data on (a test asserts none appears).
- KRS has no single status field. `status` is derived from the register's liquidation, bankruptcy, dissolution and restructuring entries (`status_basis` says so; `procedures` lists the entries found). `no_liquidation_bankruptcy_or_dissolution_entries` means exactly that, not a guarantee the entity is trading. Entities struck off the register were not observed and their behaviour is unverified.
- Names and addresses are in Polish, upper case, as the register states them; `legal_name_native` repeats `legal_name`.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "krs-pl": {
      "url": "https://gateway.pipeworx.io/krs-pl/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/krs-pl/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 2018+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/krs_company_get \
  -H 'Content-Type: application/json' \
  -d '{"krs":"0000019193"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/krs_company_get`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-krs-pl` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/krs-pl/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "krs-pl": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-krs-pl"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Krs Pl data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
