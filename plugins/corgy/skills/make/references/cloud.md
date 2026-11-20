# Recording from the cloud

For an agent with no shell on the user's Mac. `start_recording_run` sends the
sheet to the server; Corgy for Mac, signed in to the same account with
**Settings → Agents → Accept recordings from assistants** on, picks it up within
seconds, asks the person at the Mac, and records it in Corgy's stage browser. Steps 1 to 3 of the skill are unchanged; this replaces step 4.

## The sheet

The same plan, in a narrower shape - the server refuses anything else and says
which scene is wrong:

```json
{
  "title": "Dynamic pages on Stormkit",
  "project": "<id from list_projects>",
  "scenes": [
    { "kind": "card", "title": "Dynamic pages", "subtitle": "One rule, any URL",
      "call_to_action": "stormkit.io" },
    { "name": "The result", "startURL": "https://sample.stormkit.dev/products/6",
      "prompt": "Wait 2000ms\nScroll down 400" },
    { "kind": "hold", "name": "Read the rule", "seconds": 4, "freezeOn": "redirects.json" },
    { "name": "Any URL works", "prompt": "Click \"Products\"\nWait 2000ms\nClick \"#121\"" }
  ]
}
```

- **Public web pages only.** `startURL` and anything a prompt opens must be a
  public `http(s)` address: never `localhost`, a private network, a file or the
  browser's own pages. A local development demo needs an agent with a shell.
- **No other applications.** Everything happens in the stage browser.
- **Cards, not HTML.** A title card is `kind: "card"` with plain-text `title`,
  `subtitle` and `call_to_action`, drawn by the app. There are no animation
  pages.
- **At most 6 takes and cards**, prompts of at most 1,500 characters.
- No `unattended` or `stage` needed: a cloud run always records on its own once
  the person allows it, and every take comes out 16:9, the page alone.
- `requested_by` is your own name (ChatGPT, Claude). The person is shown it.

## Driving the takes

From Corgy for Mac 0.1.20, steer product takes yourself instead of prompting
them: give each one `"driven": true` and a one-sentence `prompt` saying what
it shows. Cards stay as they are.

```json
{ "name": "Pricing", "startURL": "https://acme.com", "driven": true, "prompt": "Shows the pricing tiers" }
```

A start refused with `device_needs_attention` saying the Mac cannot drive
takes is an older Corgy: ask the user to update it, or send prompts instead.

Once `get_recording_run` says `recording`, call `drive_recording_run` with
`look`, then one `step` at a time, reading each answer before the next:

- `command.result.outcome` — `done`, `unchanged`, `failed` (with `error`; the
  take is still recording), `looked`, `ended`
- `command.result.look` — the page's `url` and the `controls` in view, as
  `role` and `label`: use the label as `target` exactly
- the screenshot, as an image beside the answer
- `command.state` `pending` or `taken` — the Mac has not answered yet: call
  again with `command_id`
- `run.state` no longer `recording` — the take ended without an answer; follow
  `get_recording_run` from here. `abort` is never answered, so it always ends
  this way.

`end` when the scene's last screen is up, after a `wait` of about 1000 ms.
With more takes after it, `look` again: the answer comes once the next take
starts recording. How to drive well - every step filmed, thinking cut, abort
rather than walk back, never type a password - is in `driving.md`.

Signed-in demos: the same stage as local runs, so the same rule as step 3 of
the skill - ask the user to sign in through **Settings → Agents → Open the stage
browser** and wait for their answer before `start_recording_run`. A run that
comes back on a sign-in page is a sign-in for the user, not a retry.

## Following it

Poll `get_recording_run` every 10 to 15 seconds. `state` goes `queued` →
`claimed` → `asking` (the person is deciding) → `recording` → one of:

- **`done`** — `video_id` is the video every take went into. Go on to step 5.
- **`refused`** — the person declined. Stop and say so.
- **`failed`** — `detail` says why; the takes before the failure are in
  `video_id`. `code` `device_lost` or `app_restarted` means the Mac went away.
- **`cancelled`**, **`expired`** — `code` says why (`not_claimed`: no Mac took
  it within two minutes).

A refused start says why in `code`: `no_device` (Corgy for Mac is not open with
Accept recordings from assistants on - ask the user to turn it on),
`device_needs_attention`, `limit_queued`, `limit_daily`, `invalid_sheet`,
`blocked_url`.

`cancel_recording_run` stops a run; the Mac stops within about 10 seconds.

## Re-recording

A new run with only the takes still needed, `into_video_id` set to the video,
and each retried scene's `retry_of` set to its `take_id` from the first run's
`sheet`. The takes land at the end of the video; move them with
`reorder_clips` if they belong earlier.
