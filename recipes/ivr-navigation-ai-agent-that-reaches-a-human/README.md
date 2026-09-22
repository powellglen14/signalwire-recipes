# IVR navigator: an AI agent that works through phone menus to reach a human

> An IVR navigation demo built to show an AI agent dialing into an unknown phone tree, listening, and pressing the digits that serve a goal, without speaking. When it reaches a human it hands off with the full path it took.

**Scenario:** reach a human in the pharmacy department of a phone tree nobody has mapped

## What this demonstrates

The model's only job is classification: that was a menu, that was hold music,
that was a human, that was a beep. Everything else is code. A scorer picks the
digit that serves the goal. The goal lives in `global_data` and never in the
prompt. A verbal policy decides the rare cases where anything may be spoken,
and handlers own the loop detection, the budgets and the terminal states. The
repository states the design test plainly. Replace the model with a component
that transcribes menus and detects humans, and the navigator still works.

The agent is configured as an outbound answerer with `wait_for_user`, so it
never speaks first when the far end picks up. Each menu it hears becomes a
fingerprint, so a loop is caught the second time round. When a human answers,
the handler hands off. That is a `swml_transfer` to a conversation agent
carrying the summary, or a `connect` to a phone or SIP destination, with the
breadcrumb path delivered either way. A test rig runs the whole flow offline
with no account.

## How it works

Seven recipes across a few flat modules. `agent.py` is an `AgentBase` with one
context and one step, whose criteria is "a terminal tool fired". `tools.py`
holds the tools the model classifies with. Each returns a `FunctionResult`
that updates the `nav` state in `global_data`. When a digit is due, the result
also runs a one-verb SWML document with `send_digits`, paced with the schema's
`W` and `w` pauses. `record_call` is added as a post-answer verb so the path is
auditable. `nav_engine.py` scores digits against the goal, and
`verbal_policy.py` is the one gate through which speech can happen.

## Limitations

This is a clone-and-own agent, not a snippet, and it needs something to place
the outbound call and hand it the goal. The repository has no live deployment
to try; the offline test rig is the way in.

Phone trees change. A path that worked yesterday is a hypothesis today, which
is why the agent listens rather than replaying a script.

## What to change first

The goal specification. `goal_spec.py` is where "reach a human in pharmacy"
becomes something the scorer can rank digits against. The listening, the
pressing and the hand-off do not change with the goal.
