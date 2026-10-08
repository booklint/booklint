# booklint

Deterministic spending guardrails for agents. Set limits; booklint checks every action against them and returns a verdict — STOP, WARN or MATCH. Hotel bookings, purchases, subscriptions, invoices, quotes, budgets and data quality. Trading books against rulebooks too. Free key in one call, no card.

## Try it (no key, no sign-up)

```bash
curl -sS -X POST https://booklint.com/v1/check \
  -H 'content-type: application/json' \
  -d '{"sample":"spend"}'
```

Returns **FLAGGED**: hotel 640 USD above 600 USD per-action cap (STOP), day total 1620 USD above 1500 USD daily cap (STOP), rail booking not refundable (STOP).

## Install

**Claude Code**

```bash
claude mcp add --transport http booklint https://booklint.com/mcp
```

**Cursor** — add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "booklint": { "url": "https://booklint.com/mcp" }
  }
}
```

**Claude Desktop** — add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "booklint": { "url": "https://booklint.com/mcp" }
  }
}
```

8 MCP tools: `get_sample`, `check_book`, `get_free_key`, `check_invoice`, `check_quote_math`, `check_budget_variance`, `check_accounts_tieout`, `check_data_quality`. Streamable HTTP, stateless. Free key at `POST /v1/keys {"plan":"free"}`, no card, no email. Agent calls `get_free_key` in-session. Full OpenAPI at [`/openapi.json`](https://booklint.com/openapi.json).
