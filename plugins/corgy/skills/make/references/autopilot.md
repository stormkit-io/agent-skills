# A sheet that records itself

The demo skill's sheet (`../../demo/references/sheet.md`), plus what a run with
nobody at the machine needs.

## The extra fields

| Field | Where | Notes |
| --- | --- | --- |
| `unattended` | top level | `true` records every take back to back without asking, and writes the report. Without it the sheet only opens, for a person to record. |
| `stage` | top level | `{ "browser": "Google Chrome", "width": 1440, "height": 900 }`, every field optional; `{}` is Chrome at 1440×900. Each take runs in a dedicated Chrome with its own profile, placed in the middle of the main display. Smaller than 640×400, or a browser other than Chrome, is refused. Only applies to unattended runs. |
| `pageOnly` | per take | `true` records only the page, without Chrome's tabs and address bar. Use it on title cards and animations; leave it off on product footage, where the address bar shows the URL is real. Falls back to the whole window if the page cannot be found. |
| `startURL` | per take | Loaded with the camera off before the take starts, then a short settle. `file://` works, which is how animations are recorded. Leave it out to start from wherever the last take ended. |

What the app does that you do not have to:

- skips the rehearsal — a take runs once
- records at high quality, the stage window only
- puts every take after the first into the first one's video, end to end
- names the video after the sheet's `title`
- closes the stage browser when the run ends, however it ends; its profile
  (and so its sign-ins) stays in `~/Library/Application Support/Corgy/Stage/Chrome`

Holds are not recorded. They stay in the sheet as the plan for step 6, where
you add them through the MCP server.

## Who may start one

An unattended sheet only records when all of these hold, and is refused with a
reason otherwise:

- **Let agents record** is on in Corgy → Settings → Agents
  (`defaults read ai.corgy.recorder agentsMayRecord` prints `1`). From Corgy
  0.1.14, a sheet that arrives while it is off asks the person once - Allow
  turns it on and the run starts, Not now writes a `refused` report. A
  downloaded sheet is refused without asking
- the sheet was opened from outside the app — `open -a Corgy <file>`. Dropped
  on the window or picked in the Open panel, it only shows the takes and waits
  for Record
- the file was not downloaded (no quarantine mark): a sheet that came from the
  web never records itself
- Screen Recording and Accessibility are granted, and the app is signed in
- no other run is going

## The report

`<sheet name>.report.json`, beside the sheet. Written fresh every time the sheet
is opened, rewritten after every take.

```json
{
  "runID": "38F38AAE-…",
  "startedAt": "2026-10-02T20:40:14Z",
  "title": "Stage smoke test",
  "state": "done",
  "videoID": "beb665c4-…",
  "videoURL": "https://app.corgy.ai/videos/beb665c4-…",
  "takes": [
    { "scene": 0, "name": "The landing page", "state": "recorded",
      "recordingID": "beb665c4-…", "atMs": 0, "durationMs": 7302,
      "steps": ["scrolls down slowly through the page", "…"] },
    { "scene": 2, "name": "Pricing", "state": "recorded",
      "recordingID": "2eaa511b-…", "atMs": 7463, "durationMs": 3107,
      "steps": ["clicks Pricing in the top navigation"] }
  ]
}
```

| Field | Notes |
| --- | --- |
| `state` | `recording`, then `done`, `failed` or `refused`. Only those last three end a run. |
| `error` | Why it failed or was refused, naming the take. |
| `videoID` | The video the takes went into — the first take's recording. Present as soon as one take is in, so a failed run still says where its good takes are. |
| `takes[].scene` | Index into the sheet's `scenes`, holds counted. |
| `takes[].state` | `pending`, `recording`, `recorded` or `failed`. |
| `takes[].recordingID` | The take's own recording, for `insert_recording`. |
| `takes[].atMs` | Where the take landed in the video, as the server placed it. |
| `takes[].steps` | What the take did, in the planner's words. |

A report stuck on `recording` with nothing changing for several minutes means
Corgy crashed; quitting it normally marks the run failed. Ask the user to
check the app.

## Waiting for it

```sh
R=/tmp/<demo-name>.report.json
until python3 -c "import json,sys; sys.exit(json.load(open('$R'))['state'] not in ('done','failed','refused'))" 2>/dev/null; do sleep 3; done
cat "$R"
```

Run it in the background, with a timeout of a few minutes per take.
