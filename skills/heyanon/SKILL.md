---
name: heyanon
description: Use HeyAnon MCP for blockchain and crypto operations, wallet/portfolio queries, DeFi, CEX orders, token transfers, protocol research and task status, even when HeyAnon is not named. Supports requests in any language. Exclude standalone smart-contract coding/auditing and explanations needing no live data.
---

# HeyAnon

Use the connected `heyanon` MCP tools and their schemas. The API key is the
plugin's `HEYANON_API_KEY` value and travels only as the `X-API-Key` header of
the MCP connection. Never read a key into model context, chat or tool
arguments; the tools take no key argument. If tools answer "API key not set"
or an HTTP 401, the user re-enters the key in the plugin settings (Cursor:
plugin page, Setup, Edit Values; Grok Bot: the app's settings in Connect apps),
never in the chat.

## Tools and requests

- Use `wallet_list` for addresses and the `portfolio_*` readers for account data.
- Use `projects` when protocol/network support is unclear; reuse its results.
- Use `ask_gemma({text})` for crypto research. It returns the answer directly
  and keeps its own conversation.
- Use `ask_anon({text, wallets})` for requested actions, automations or questions
  needing the Anon agent. It can execute transactions and returns a task ID.
  Optional attachments use `files: [{url, mimetype}]`.
- Use `transfer_native` or `transfer_asset` only for a plain send the user spelled
  out completely: network, sending wallet, recipient, asset and exact amount
  (an integer string in the smallest unit). Resolve a named contact with
  `address_book_list` and confirm the address with the user first. Swaps,
  bridging, staking and automations go through `ask_anon`.

Before `ask_anon`, select addresses from `wallet_list`: at least one, at most
one per type (EVM, Solana, TON). Match the wallet the user names by display
name, label or address, skip archived wallets, and use a previously chosen
wallet when there is one; if several fit and none was chosen, ask. Reuse
addresses for follow-ups.

Anon keeps no context between messages: every `ask_anon` text starts from a
clean slate and must be self-contained. Repeat the wallet, network, token,
amount, recipients, protocol, constraints and earlier choices each time; a
follow-up such as "and on Base?" will not work. Keep linked steps (swap then
deposit the proceeds) together in one message. Resolve missing facts through
readers/context; ask for essential unresolved choices and retain choices the
user delegates. Do not substitute an explicitly requested provider or chain.
Tool approval does not authorize actions beyond the user's request.

Anon handles one request at a time: wait for each task to finish before
sending another, including from subagents. If `ask_anon` answers that Anon is
still processing, poll `background_task` for the running task (use `abort`
only when it is really stuck), then send the message again.

## Results

Read `background_task({id})` using the returned ID: `pending` means wait;
`completed` means read `result`; `failed` means report the result/logs.
Completion can contain a clarification or partial failure. Report the actual
answer and returned transaction links; an ID alone proves no execution.

If submission times out or returns an invalid ID, inspect `background_task_list`
before retrying. Never repeat a transaction request just to obtain its result.
`background_task_delete({id})` deletes a record; it does not establish cancellation.
Scheduled automations use `scheduled_task_list` and `scheduled_task_get`.

## Waiting

Poll `background_task({id})` about every five seconds until `status` leaves
`pending`. A long wait can be handed to a background worker together with the
existing task ID; the worker only polls, it never submits anything. Stopping
to wait does not cancel the backend task; resume reading the same ID. Do not
use account-wide `abort` or `isProcessing` for one task.

## Confirmation

`ask_anon`, `transfer_native`, `transfer_asset`, `abort`,
`background_task_delete` and `scheduled_task_delete` change the account or move
real funds and cannot be undone. Before each such call, show the user the
exact action, asset, amount, wallet, recipient and network and wait for their
explicit confirmation; when the client prompts for the call itself, that
prompt is the confirmation. Never proceed on an ambiguous or missing detail,
and report transaction hashes and explorer links exactly as returned.
