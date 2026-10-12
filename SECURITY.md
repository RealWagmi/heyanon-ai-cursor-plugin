# Security Policy

## Reporting a vulnerability

If you find a security problem in the HeyAnon MCP server or in this plugin,
report it responsibly. Do not open a public GitHub issue for vulnerabilities.

Contact: [info@heyanon.ai](mailto:info@heyanon.ai) with "security" in the subject. We acknowledge
reports and work on a fix before any public disclosure.

## Scope

- The remote MCP server at `https://api.heyanon.ai/mcp`
- The plugin configuration and skill files in this repository

## Practices

- All traffic to the MCP server uses HTTPS.
- Authentication is an API key sent as the `X-API-Key` header of the MCP
  connection; tools take no key argument and the key is never requested in chat.
- MCP tool annotations (`readOnlyHint`, `destructiveHint`) tell clients which
  tools only read and which change the account or move funds.
- No secrets or API keys are stored in this repository.
