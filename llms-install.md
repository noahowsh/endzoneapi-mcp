# Installing the EndzoneAPI MCP server

EndzoneAPI is a hosted (remote) MCP server over Streamable HTTP. There is nothing to clone, build or run locally.

## 1. Get an API key

Ask the user for their EndzoneAPI API key. If they do not have one, they can create one free (500 credits, no card) at https://endzoneapi.com/signup. Connecting and listing tools works without a key; calling a tool needs one.

## 2. Add the server

For Cline, add this to `cline_mcp_settings.json`, replacing YOUR_API_KEY:

```json
{
  "mcpServers": {
    "endzoneapi": {
      "type": "streamableHttp",
      "url": "https://mcp.endzoneapi.com/mcp",
      "headers": { "x-api-key": "YOUR_API_KEY" },
      "disabled": false
    }
  }
}
```

If the client cannot send headers, put the key in the URL instead: `https://mcp.endzoneapi.com/mcp?key=YOUR_API_KEY`.

## 3. Check it works

Call `list_teams`. It costs 1 credit and returns every team, which confirms the key and connection.

## Notes

- Every tool is read-only.
- Each call costs credits (the tool descriptions say how many). An answer with no data is refunded.
- Errors come back as JSON with a plain-language message; an out-of-credits error includes an upgrade link.
