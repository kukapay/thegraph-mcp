# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

An MCP (Model Context Protocol) server that exposes The Graph's indexed blockchain data to AI agents. Built with Python's FastMCP framework, it provides three tools: `searchSubgraphs` (find subgraphs by name/description), `getSubgraphSchema` (introspect a subgraph's GraphQL schema), and `querySubgraph` (execute arbitrary GraphQL queries against a subgraph).

## Commands

```bash
# Run the MCP server
uv run main.py

# Install/sync dependencies
uv sync
```

There is no test suite or linter configured.

## Architecture

The entire server lives in `main.py` (~115 lines):

- **FastMCP server** instance exposes tools via `@mcp.tool()` decorators
- **`searchSubgraphs(searchQuery)`** — uses the Network Subgraph's `subgraphMetadataSearch` full-text search to find subgraphs by name/description. Returns a list sorted by signal (highest first) with subgraph IDs, display names, networks, signal amounts, descriptions, and full GraphQL schemas. Filters out inactive subgraphs (no current version). Schemas are included so agents can go directly from discovery to querying.
- **`getSubgraphSchema(subgraphId, asText)`** — fetches a subgraph's schema via GraphQL introspection. When `asText=True`, the helper `json_to_graphql_schema()` converts the JSON introspection result into human-readable GraphQL SDL.
- **`querySubgraph(subgraphId, query)`** — posts an arbitrary GraphQL query to The Graph gateway and returns the JSON response.
- All API calls go through `httpx.AsyncClient` to `https://gateway.thegraph.com/api/{API_KEY}/subgraphs/id/{subgraphId}` with a 10-second timeout.
- Authentication is via the `THEGRAPH_API_KEY` environment variable (loaded from `.env` by python-dotenv).

## Dependencies

Managed by **uv** (lock file: `uv.lock`). Python >=3.13 required.

- `mcp[cli]` — MCP server framework
- `httpx` — async HTTP client
- `dotenv` — .env file loading
