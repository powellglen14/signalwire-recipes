# Competitor alerts on live calls: real-time agent assist from an AI listener

> A sales-call demo built to show real-time agent assist: an AI listener rides along on a live call, notices when the customer names a competitor, and posts the alert to your server while the sentence is still being spoken. It never speaks into the call.

**Scenario:** a sales rep is on a call and an alert lands on their screen the moment the customer says a rival's name

## What this demonstrates

The `ai_sidecar` SWML verb attaches a listening AI to a call it does not own.
The listener hears both legs and treats the remote party as the customer. It
has one tool, `flag_competitor`, which it calls when a name from a list is
spoken. The tool handler records the mention and forwards it to a webhook of
yours. The listener's own speech channel is never used.

The demo's offline run is the argument. `run_demo.py` replays a transcript
against the detector and prints each alert beside the words that triggered
it. The alert lands with nine words of the sentence still to come. The demo
counts words rather than milliseconds on purpose: a position in the transcript
is an observable fact, and a duration is a performance claim. A deployment is
live at https://sidecar-demo.signalwire.me. Its routes let you feed a line of
speech in and read the alerts back without a call.

## How it works

The host is `SWMLService`, not `AgentBase`, because the listener does not run
the conversation. The document is `answer`, then `ai_sidecar`, then `hangup`.
The verb carries a prompt that names the competitors and a `direction`
covering both legs. It sets `customer_role` to the remote caller, adds
recognition hints for the names, and points `web_hook_url` at the tool.
Lifecycle and transcription events arrive at `/sidecar/events`, and the
sidecar's LLM calls `flag_competitor` at `/sidecar/swaig`. A `/simulate` route runs the same detector path on typed
text, and `/timeline` shows transcript and alerts on one clock. No SignalWire
credentials are needed to run it. The platform calls the service; the service
never calls out.

## Limitations

`ai_sidecar` requires `signalwire-sdk` 3.4.1 or newer. It does not exist in
the 3.0.1 the recipes here are verified against, which is why the sidecar
recipe is planned rather than written. The larger `signalwire-demos/sidecar`
demo adds a rep workspace, a live monitor and call records; this is the small
one that goes with the quickstart. The repository carries no licence file at
the time of writing, which is its owner's to add.

## What to change first

`competitors.txt`. The names are the whole domain. The verb, the tool and the
event routes are the same for any list of words worth an alert.
