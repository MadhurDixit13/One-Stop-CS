# Model Context Protocol (MCP)

[![MCP](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-purple)](https://modelcontextprotocol.io/)

> An open protocol for safely connecting AI models to tools, data, and workflows. MCP standardizes how LLMs discover, call, and reason about tools and data sources.

## What is MCP?
- **Purpose**: Define a vendor-neutral way for LLMs to talk to external tools and data.
- **Model-agnostic**: Works with any LLM/provider.
- **Transport-agnostic**: JSON-RPC over stdio, WebSocket, or HTTP.
- **Capabilities**: Tools, prompts, resources, file systems, events, and streaming.

## Core Concepts
- **Server**: Exposes capabilities (tools, resources, prompts) via MCP.
- **Client**: Hosts the model and orchestrates tool calls (e.g., IDE, agent, chat app).
- **Capabilities**:
  - Tools: callable actions with JSON schemas
  - Resources: read-only data (e.g., docs, DB queries)
  - Prompts: server-provided prompt templates
  - Files: virtual FS access (optional)
  - Events/Streaming: progress, partial outputs

## Minimal Server (Node.js)
```ts
// package.json: { type: "module" }
import { createServer } from "@modelcontextprotocol/sdk/server";
import { z } from "zod";

const server = createServer({ name: "demo-server", version: "0.1.0" });

// Define a tool with schema
const EchoInput = z.object({ text: z.string() });
server.tool("echo", {
  description: "Echo input text",
  schema: EchoInput,
  handler: async ({ text }) => ({ text }),
});

// Optional: expose a resource
server.resource("docs:intro", async () => ({
  mimeType: "text/markdown",
  data: Buffer.from("# Intro\nThis is a demo resource.")
}));

// Start (stdio)
server.start();
```

Run:
```bash
npm i @modelcontextprotocol/sdk zod
node server.mjs
```

## Minimal Client (TypeScript)
```ts
import { StdioClient } from "@modelcontextprotocol/sdk/client";

const client = new StdioClient({ command: "node", args: ["server.mjs"] });
await client.connect();

// Discover capabilities
const caps = await client.listCapabilities();
console.log(caps);

// Call tool
const result = await client.callTool("echo", { text: "hello" });
console.log(result);

// Read resource
const doc = await client.readResource("docs:intro");
console.log(doc.toString());
```

## JSON-RPC Shapes
- initialize → capabilities
- tools/call → result | error
- resources/read → bytes + mime
- prompts/list, prompts/get → template metadata
- events/subscribe → stream of progress/events

## Validation & Schemas
- Prefer zod/JSON Schema for inputs/outputs.
- Validate all inputs server-side.
- Return actionable error codes/messages.

## Security Considerations
- Principle of least privilege: expose minimal tools/data.
- Input sanitation and output encoding.
- AuthN/AuthZ on transports (e.g., tokens over WS/HTTP).
- Rate limiting and timeouts per call.
- Sandboxed side effects (FS/network) and audit logging.

## Testing
```bash
# Contract tests (Node)
npx vitest run
# Schema testing
npx tsx scripts/validate-schemas.ts
```

## Observability
- Log request/response IDs and durations.
- Emit progress events for long-running tasks.
- Capture tool error rates and latency histograms.

## Patterns
- Toolbox servers: many small tools, single server.
- Facade server: wrap legacy APIs with clean schemas.
- Resource-first: prefer read-only resources before imperative tools.
- Idempotent by default: safe retries on network errors.

## Compatibility
- Works alongside OpenAI function calling, Anthropic Tools, Azure Functions—MCP provides a stable, cross-vendor layer.

## References
- Spec and guide: modelcontextprotocol.io
- SDKs: `@modelcontextprotocol/sdk` (TS), community ports (Python, Go)
- Examples: `https://github.com/modelcontextprotocol/examples`
