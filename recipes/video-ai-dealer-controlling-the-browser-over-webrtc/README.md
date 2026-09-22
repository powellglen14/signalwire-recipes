# Video blackjack dealer: a voice AI agent that controls the browser

> A blackjack game built to show a video AI agent controlling a web page in real time over WebRTC. Stepped conversation contexts run the hand, every card and chip change reaches the page as an event, and the dealer's voice is chosen on the page.

**Scenario:** blackjack against a dealer with a face and a voice you chose, in a browser tab

## What this demonstrates

A voice AI agent that drives a browser interface rather than describing one.
You press play. The page mints a guest token and dials the agent's Fabric
address with the Browser SDK, and a dealer with a video face greets you. Each
hand runs through three conversation steps: `betting`, `playing` and
`hand_complete`. The tools offered to the model change with the step, so the
dealer cannot be talked into paying out mid-hand. The game logic lives in the
tool handlers; the model narrates what the code decided.

Every state change reaches the page as a `user_event`: cards dealt, chips
moved, hand resolved. The page redraws from those events and never asks the
agent what happened. Before the call, the page offers a voice picker. The
choice travels as a query parameter, and the agent applies it at render time
after checking it against an allowlist. That allowlist is built from the same
voice catalogue the picker shows. A deployment is live at
https://blackjack.signalwire.me.

## How it works

Seven recipes in one process. An `AgentBase` serves the SWML, the tool
webhooks and the static page. On startup it registers itself as a Fabric SWML
handler and reads back its address, so no dashboard step is needed. A
`/get_token` route mints a guest token for that one address. `on_swml_request`
runs per call: it sets the avatar's idle and talking video files from the
detected host and applies the requested voice. `define_context` with three
steps, each with `set_functions` and `set_step_criteria`, shapes what the model
can do. The handlers move the step with `swml_change_step` once the code has
settled the hand. `swml_user_event` carries fifteen event types to the page,
and the page subscribes to `user_event` on the call.

## Limitations

This is a clone-and-own application: a single Python process, a web client,
and a Dokku deployment. The repository carries no licence file at the time of
writing, which is the repository owner's to add.

The voice picker only offers voices the platform serves. A voice that is
staged but not live falls back mid-call, which is why the allowlist exists.

## What to change first

The game. `betting`, `playing` and `hand_complete` are three steps and a
handful of handlers. A different game is a different set of steps and events;
the page, the token route and the avatar do not change.
