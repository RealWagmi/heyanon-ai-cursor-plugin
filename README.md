# HeyAnon MCP plugin

Connect Cursor or Grok Bot to [HeyAnon](https://heyanon.ai/): wallets, portfolios,
DeFi and CEX actions through Anon, the HeyAnon on-chain agent; Gemma crypto
research; token transfers and scheduled automations. The plugin registers the
remote HeyAnon MCP server and ships an Agent Skill that teaches wallet
selection, self-contained Anon requests, background-task polling and
confirmation rules.

## Tools

| Tool | Key | What it does |
| --- | --- | --- |
| `ping`, `get_api_key_setup_url` | no | Server check; where to create an API key |
| `wallet_list` | yes | Your wallets: name, type (evm, solana, ton), address, labels |
| `address_book_list` | yes | Saved recipient addresses |
| `projects` | yes | Supported protocols and networks |
| `portfolio_tokens`, `portfolio_transactions`, `portfolio_defi`, `portfolio_cex` | yes | Portfolio readers per wallet |
| `ask_gemma` | yes | Crypto research; answers directly |
| `ask_anon` | yes | **Action.** Message to Anon: balances, swaps, bridging, lending, staking, CEX, automations. Returns a background task ID |
| `background_task`, `background_task_list` | yes | Read a task's status, result and logs |
| `scheduled_task_list`, `scheduled_task_get` | yes | Automations created through Anon |
| `transfer_native`, `transfer_asset` | yes | **Moves funds.** Send the native coin or a token to an address; irreversible |
| `abort`, `background_task_delete`, `scheduled_task_delete` | yes | **Changes state.** Stop processing; delete a task record or an automation |
| `isProcessing` | yes | Whether Anon is still busy with the previous request |

Read-only tools are marked as such with MCP tool annotations; `ask_anon`, the
transfer tools, `abort` and the delete tools are marked destructive. Anon can
execute real transactions from the attached wallets, so the skill tells the
agent to show the exact action and wait for your explicit confirmation before
every such call. Client prompts add a second gate: Cursor asks before MCP tool
calls by default, Grok Bot reviews tool calls and lets you add "Ask first"
rules in its settings.

## Setup

### Grok Bot

Open **Connect apps**, find **HeyAnon MCP** and choose **Add**. Enter your API
key in the secure field the app asks for; it is stored by Grok Bot and sent only
as the `X-API-Key` header of the MCP connection. Get the key at
https://heyanon.ai/ (account settings, API keys). Do not paste it into the chat.

### Cursor

Install **HeyAnon MCP** from the Cursor Marketplace, open the plugin, then
**Setup → Edit Values** and paste your HeyAnon API key. The skill appears in
**Customize**.

### Manual setup

Add to `.cursor/mcp.json` in your project (or `~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "heyanon": {
      "url": "https://api.heyanon.ai/mcp",
      "headers": {
        "X-API-Key": "YOUR_API_KEY"
      }
    }
  }
}
```

Codex, Claude Code, Hermes and OpenClaw users can use
[HeyAnon Connect](https://github.com/RealWagmi/heyanon-connect) instead; it
installs the same connection and skill with a local key page.

## Examples

**"What are my balances on Base?"** The agent takes your EVM address from
`wallet_list`, sends one self-contained question through `ask_anon`, polls
`background_task` until it completes and reports the answer.

**"Swap 10 USDC to ETH on Base from my main wallet"** The agent confirms the
exact action, sends it through `ask_anon`, waits for the task and reports the
transaction links from the result.

**"Send 0.1 ETH to my work address on Arbitrum"** The agent resolves the
contact with `address_book_list`, confirms the address and amount with you, and
calls `transfer_native` once.

**"Any news about Aave this week?"** Research goes to `ask_gemma`, which answers
directly.

Anon keeps no context between messages: every request must be self-contained,
and Anon handles one request at a time.

## Privacy and security

- The API key is stored by the client (Cursor or Grok Bot) and sent only to
  `https://api.heyanon.ai/mcp` over HTTPS as the `X-API-Key` header.
- No secrets are stored in this repository, and nothing runs locally: the plugin
  is configuration and a skill for a remote server.
- [HeyAnon privacy policy](https://heyanon.ai/privacy-policy)
- Security reports: see [SECURITY.md](SECURITY.md)

## Support

Questions and feedback: [info@heyanon.ai](mailto:info@heyanon.ai) or https://heyanon.ai/.

## License

MIT
