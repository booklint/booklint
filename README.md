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

## Use it from code

Each example connects to `https://booklint.com/mcp` with no key and asks the agent to run the booking sample, which returns real STOP results and is never counted against the free caps. Without a key, an agent gets 5 own checks per client address per UTC day. A free key (`get_free_key`, no card) gives 20 a day; send it as `Authorization: Bearer vdk_...`.

**LangChain** (`pip install langchain langchain-anthropic langchain-mcp-adapters`)

```python
import asyncio

from langchain.agents import create_agent
from langchain_mcp_adapters.client import MultiServerMCPClient


async def main():
    client = MultiServerMCPClient(
        {"booklint": {"transport": "http", "url": "https://booklint.com/mcp"}}
    )
    tools = await client.get_tools()
    agent = create_agent("anthropic:claude-opus-5-5", tools)
    result = await agent.ainvoke(
        {"messages": "Call check_book with sample='booking' and list every STOP result."}
    )
    print(result["messages"][-1].content)


asyncio.run(main())
```

**Anthropic SDK**, MCP connector (`pip install anthropic`)

```python
import anthropic

client = anthropic.Anthropic()
response = client.beta.messages.create(
    model="claude-opus-5-5",
    max_tokens=4096,
    betas=["mcp-client-2025-11-20"],
    mcp_servers=[{"type": "url", "url": "https://booklint.com/mcp", "name": "booklint"}],
    tools=[{"type": "mcp_toolset", "mcp_server_name": "booklint"}],
    messages=[{"role": "user",
               "content": "Call check_book with sample='booking' and list every STOP result."}],
)
for block in response.content:
    if block.type == "text":
        print(block.text)
```

The connector calls booklint from Anthropic's servers, so keyless calls share one client address with other connector users. To get your own cap, add `"authorization_token": "vdk_..."` (a free key) to the `mcp_servers` entry.

**CrewAI** (`pip install 'crewai[anthropic]' 'crewai-tools[mcp]'`)

```python
from crewai import LLM, Agent, Crew, Task

agent = Agent(
    llm=LLM(model="anthropic/claude-opus-5-5", max_tokens=4096),
    role="Booking checker",
    goal="Check every booking against the owner's limits before it is confirmed",
    backstory="Calls booklint before confirming a booking, spend or subscription.",
    mcps=["https://booklint.com/mcp"],
)
task = Task(
    description="Call check_book with sample='booking' and list every STOP result.",
    expected_output="Each STOP result with its rule and the figure found.",
    agent=agent,
)
print(Crew(agents=[agent], tasks=[task]).kickoff())
```

8 MCP tools: `get_sample`, `check_book`, `get_free_key`, `check_invoice`, `check_quote_math`, `check_budget_variance`, `check_accounts_tieout`, `check_data_quality`. Streamable HTTP, stateless. Free key at `POST /v1/keys {"plan":"free"}`, no card, no email. Agent calls `get_free_key` in-session. Full OpenAPI at [`/openapi.json`](https://booklint.com/openapi.json).

## Run it locally

The free rule set also ships as a local Python package (MIT, Python 3.11+, no dependencies): per-action and daily
spend caps, the total price cap and fees above the headline price of an offer, the trading floors, and five finance
tools. It runs on your machine with no network, no key and no caps; rules outside the free set report
`not_in_plan` and run on the hosted Desk plan. Download the wheel from the
[latest release](https://github.com/booklint/booklint/releases/latest), then:

```bash
pip install booklint-0.1.0-py3-none-any.whl
booklint check request.json      # the same body as POST https://booklint.com/v1/check
booklint mcp                     # a local MCP server over stdio
```

MCP client config for the local server: `{"mcpServers": {"booklint": {"command": "booklint", "args": ["mcp"]}}}`
