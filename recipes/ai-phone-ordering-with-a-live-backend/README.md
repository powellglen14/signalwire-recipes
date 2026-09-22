# AI phone ordering: a voice agent working with a live backend in real time

> A coffee shop ordering demo built to show a voice AI agent working with a real backend in real time. Every item is a tool call into the order system, nothing exists until a handler has recorded it, and the page shows each call as it happens.

**Scenario:** a coffee order by phone or browser, with a live ticket drawn from what the backend recorded and the tool calls listed under it

## What this demonstrates

The hard problem in a voice ordering agent is not transcription. It is that a
model will confirm an order it never recorded. This build is arranged around
that. The order is whatever the tool handlers wrote, and the ticket on the
page is drawn from that state. The tool calls are printed under the ticket, so
you can see the difference between what the model said and what the code did.

Seven tools do the work. `add_to_order` resolves speech to a catalogue item
and handles sizes, quantities and ambiguity. It refuses off-menu requests from
a written table rather than a fuzzy match, because fuzzy matching once
answered "water" with "Latte". `donate` takes a fixed tier from an `enum` on
the parameter and checks it again server-side. `place_order` is idempotent, so
a repeated "that's it" returns the same order number instead of a second
order. The same agent answers a phone number and a browser call.

## How it works

Eight recipes in one `AgentBase`. It serves the SWML, the tool webhooks and the
page from one process. On startup it registers a Fabric SWML handler, and
`/get_token` mints a guest token per browser visit so the page can dial with
the Browser SDK. A phone number pointed at the same URL reaches the same
agent. Order state lives in `state.py` keyed by call id, not in `global_data`,
and the page polls `/api/state` for the ticket. Each handler emits a
`swml_user_event` the page uses to animate the line it added. A signing key,
when set, has the platform sign each webhook and the SDK verify it.

## Limitations

This is a clone-and-own application, not a snippet: a Python process, a web
client, and Docker or Dokku to run it. It pins `signalwire-sdk` 3.4.1 or newer,
ahead of the 3.0.1 the recipes here are verified against. The repository
carries no licence file at the time of writing, which is its owner's to add.

The catalogue is a coffee menu. The matching rules are written for it and will
need rewriting for a menu with different collisions.

## What to change first

`catalog.py`. The items, the off-menu table and the matcher are what make this
a coffee shop. The tools, the ticket and the token route stay as they are for
any menu.
