# Order status desk: one omnichannel AI agent for web chat and phone

> An order status demo built to show one AI agent serving web chat and phone calls, with the conversation moving between the two without starting over. It answers questions about order status and declines everything else, on purpose.

**Scenario:** a customer asks about an order in web chat, escalates to a call, and the voice agent already knows the thread

## What this demonstrates

The same `AgentBase` serves two channels. A phone number fetches its SWML. The
chat panel on the page reaches it through a gateway that forwards to the AI
Chat API, with the agent's own URL as `config_url`. The page holds a
publishable key and nothing else. The agent has one tool, an order lookup, and
a prompt that declines every other topic.

The part worth reading is the switch. A visitor typing in the chat can ask for
a call. The gateway ends the text leg and waits for its record, then places
an outbound call whose SWML URL carries a one-time nonce. When the platform
fetches that SWML, `on_swml_request` reads the nonce from the query string and
the call id from the platform's own request, and joins the two. Neither value
came from the browser, so a page cannot attach itself to someone else's call.
The reverse works too: a call can hand back to text, and the visitor can type
into a live call. A deployment is live at
https://ai-chat-demo.signalwire.me/demo/.

## How it works

Eight recipes in one FastAPI process. The SDK's `ChatGateway` is mounted at
`/chat/` and its `HandoffRouter` beside it, so the browser derives every route
from one configured URL. `client.calling.dial` places the escalation call with
`url` pointing at the agent. `client.calling.ai_message` types a visitor's
text into the live call as a user message. `client.calling.end` closes a leg
when the conversation moves. A `post_prompt` summarises each leg so the next
one can pick it up. The chat routes need an API token with the `chat` scope.
The SWML route sits behind basic auth, and the app embeds the pair in the
`config_url` it hands the gateway.

## Limitations

This is a clone-and-own application: one process plus the page it serves. It
requires `signalwire-sdk` 3.4.1 or newer for `ChatGateway` and
`HandoffRouter`, which the pinned 3.0.1 does not carry. The two chat recipes
here build the same wire by hand. The repository carries no licence file at
the time of writing, which is its owner's to add.

Handles are HMAC-signed with a key from the environment. Omit it and handles
stop verifying across a restart or a second worker.

## What to change first

The orders. `orders.json` and the one lookup tool are the whole domain. The
gateway, the handoff routes and the switch do not know what the agent is for.
