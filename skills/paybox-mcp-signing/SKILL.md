---
name: paybox-mcp-signing
description: Keep a PayBox request from stalling at the signing step on MCP connector hosts. Use when get_request returns pending_signature, when the user says the signing window never appeared, or when a request the user approved ends as denied.
license: MIT
---

# When the signing window does not come back

Your standing instructions cover the signing flow. This skill covers one gap in it on MCP connector
hosts, where the signing window is drawn from a tool response.

## Invoke this skill when

- `get_request` returns `pending_signature`, especially after `pending_approval`
- The user approved with their passkey but says no signing window appeared
- A request the user meant to sign ended as `denied` with no signature

## Why it happens

The host draws the signing window from the response of the tool call that started the request.
Polling `get_request` does not draw it again. A request that waits on `always_approve` first moves
`pending_approval` → `pending_signature` after the user approves, but by then the original window is
gone. The request waits for a signature that the user has nowhere to give, and then times out as
`denied`.

## What to do

1. Keep polling `get_request` while the status is `pending_approval`.
2. When it moves to `pending_signature`, call `reopen_signing_window` with the same `request_id`
   **once**, then tell the user the signing window is open and that they should sign there.
3. Keep polling `get_request` until it reaches a terminal status, then report the outcome.

If `reopen_signing_window` is not in your tool list, tell the user the request is waiting for their
signature in the PayBox window, and ask them to open it from the PayBox app.

## Never

- Never call `reopen_signing_window` in a loop. One call per `pending_signature` is enough, and more
  only stacks windows.
- Never start a new request to work around a stalled one. That can authorize the same spend twice.
- Never report a request as done, or as failed by the user, while it is still `pending_signature`.
