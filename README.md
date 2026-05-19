# engram-dify

[Dify](https://github.com/langgenius/dify) integration for [Engram](https://lumetra.io) — durable, explainable memory for the open-source LLM app platform.

Adds six MCP tools to Dify Agents — `store_memory`, `query_memory`, `list_memories`, `list_buckets`, `delete_memory`, `clear_memories` — backed by the hosted Engram MCP server.

## Setup

### 1. Get an Engram API key

Sign up at <https://lumetra.io> — free tier, no card. You'll see an `eng_live_…` token in your dashboard.

### 2. Configure a BYOK provider key

Engram is bring-your-own-key end-to-end for the LLM that handles extraction and synthesis. Configure one provider at <https://lumetra.io/models>. DeepSeek is what we recommend — cheap and fast. Without a provider key, every `store_memory` / `query_memory` returns HTTP 412.

### 3. Install the Dify MCP plugin

Dify 1.0+ supports MCP via a plugin. The community-maintained one is **`dify-plugin-tools-mcp_sse`** by Junjie M:

- Plugin repo: <https://github.com/junjiem/dify-plugin-tools-mcp_sse>
- Install it from your Dify console: **Plugins → Marketplace** → search for "MCP HTTP with SSE", install.

(There's also `dify-plugin-agent-mcp_sse` if you want an Agent-strategy-level integration rather than a tools-level one.)

### 4. Configure the engram MCP server inside the plugin

After the plugin is installed, open its config and add the Engram server:

```json
{
  "engram": {
    "transport": "sse",
    "url": "https://mcp.lumetra.io/mcp/sse",
    "headers": {
      "Authorization": "Bearer eng_live_..."
    },
    "timeout": 30,
    "sse_read_timeout": 600
  }
}
```

Save. The six Engram tools appear in the Dify tool catalog and can be added to any Agent / Chatflow / Workflow that consumes tools.

## Tools exposed

| Tool | What it does |
|---|---|
| `store_memory(content, bucket?)` | Save a fact to a bucket (defaults to `"default"`). |
| `query_memory(question, bucket?)` | Hybrid retrieval + synthesized answer with citations. |
| `list_memories(bucket, limit?)` | Newest-first list of memories in a bucket. |
| `list_buckets(limit?, offset?)` | Paginated list of all buckets in your tenant. |
| `delete_memory(memory_id, bucket)` | Remove a single memory. |
| `clear_memories(bucket)` | Empty a bucket. Destructive. |

## A note on Dify's MCP support

As of writing, Dify's MCP support is via community plugins rather than first-party integration. The `dify-plugin-tools-mcp_sse` plugin above is the most-maintained option and supports both SSE and Streamable HTTP. Watch [the Dify changelog](https://docs.dify.ai/) for first-party MCP integration as the platform matures.

## Manual verification

Outside Dify, confirm Engram itself is reachable with your key:

```bash
curl -s https://api.lumetra.io/v1/buckets \
  -H "authorization: Bearer eng_live_..." | head -c 300
```

You should see a JSON bucket list. If that 200s but the Dify MCP plugin shows the engram server as offline / no-tools, double-check the plugin config JSON syntax (Dify's plugin config UI is strict about commas) and that the Dify worker process can reach `mcp.lumetra.io` from inside Docker (i.e. no firewall / egress blocks).

## License

MIT — Lumetra
