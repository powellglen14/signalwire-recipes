# AI drive-thru ordering: a voice agent whose flow your code controls

> A fast-food drive-thru demo built to show a voice AI agent run by a state machine in your code. Steps scope the tools, handlers move the flow, upsell rules run as code, and a live order board updates from events.

**Scenario:** a guacamole drive-thru with an avatar and a live order board, where the code notices the combo and offers it

## What this demonstrates

An ordering agent where the application controls the model rather than the
other way round. The conversation runs through defined steps, each with its
own set of tools, and the handlers move it forward with `swml_change_step`
once the code is satisfied. Menu matching is a vector similarity over the
menu with fallbacks, not a prompt asking the model to guess. Combo detection
runs in the handler after each item and offers the upsell itself, so the
model never has to remember the rules.

The board on the page is drawn from events. Twelve event types leave the agent
as `swml_user_event`, `item_added` and `order_total` among them. The
repository's `EVENTS.md` documents every one with its fields, and it is the
best-written event contract in the demo collection. The agent has a face from
two video files and speaks with the avatar's voice. A deployment is live at
https://holyguacamole.signalwire.me.

## How it works

Eight recipes in one `AgentBase`. It serves the SWML, the tool webhooks and the
page from one process. On startup it registers a Fabric SWML handler, and it
mints a guest token per visit so the page can dial with the Browser SDK.
`on_swml_request` sets the idle and talking video files from the detected
host per call. `define_context` with stepped `valid_steps`, `set_functions`
and `set_step_criteria` shapes what the model is offered at each stage. The
barge settings are set in code: `transparent_barge` on during the order, and a
greeting the caller cannot talk over. Ten tools do the ordering, and each one
returns a `swml_user_event` action alongside its spoken response.

## Limitations

This is a clone-and-own application, one large Python file and a web client,
deployed on Dokku. The README in the repository is a long implementation
guide rather than a quick start; read it as one.

The menu matcher is tuned to this menu. A different menu keeps the shape and
needs its own tuning, and the combo rules are written for these items.

## What to change first

The menu and the combo rules. They are data and handler code at the top of
`app.py`. The steps, the events and the board do not know what is being sold.
