# Recording from the cloud

For an agent with no shell on the user's Mac. `start_recording_run` sends the
sheet to the server; Corgy for Mac, signed in to the same account with
**Settings → Agents → Accept recordings from assistants** on, picks it up within
seconds, asks the person at the Mac, and records it in a cloud stage browser of
its own. Steps 1 to 3 of the skill are unchanged; this replaces step 4.

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

Signed-in demos: the cloud stage has its own profile. If a take needs a login,
ask the user once to use **Settings → Agents → Open the cloud stage browser** and
sign in there.

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
