# The demo sheet

What you hand over instead of a prompt somebody has to paste: one JSON file
holding the whole plan. The Mac app opens on it, lists the takes, fills the box
in from whichever one is selected, and aims each take into the video the first
one created.

Corgy **0.1.4** or newer. An older copy launches and ignores the file, so the
window opens empty — if that happens, the answer is Check for Updates rather
than a different file.

## Writing one

Somewhere temporary, named for the demo:

```sh
cat > /tmp/how-corgy-works.json <<'JSON'
{
  "title": "How Corgy works",
  "claim": "A sentence becomes a narrated demo video",
  "narration": "Corgy is a demo builder. You describe the demo in a sentence…",
  "preconditions": ["Signed in at app.corgy.ai", "A demo with no narration yet"],
  "scenes": [
    {
      "name": "What came back",
      "summary": "The demos list, one demo opened, already cut into clips",
      "startState": "Chrome, signed in",
      "seconds": 18,
      "prompt": "Open Google Chrome\nNavigate to https://app.corgy.ai\nWait 3000ms"
    },
    {
      "kind": "hold",
      "name": "The hold",
      "seconds": 6,
      "freezeOn": "the timeline"
    }
  ]
}
JSON
```

| Field | Notes |
| --- | --- |
| `title` | Required. Names the demo in the app's rail. |
| `claim` | The one thing the whole demo argues. On screen while recording, which is what catches a take nobody needs. |
| `narration` | The prompt for the Voice panel, written last. Kept here so it survives the recording session between planning it and using it. |
| `preconditions` | What has to be true before the first take. Parsed and carried, not shown in the window — say them in chat as well. |
| `scenes` | Required, and at least one of them a take. |

And per scene:

| Field | Notes |
| --- | --- |
| `name` | Required. What the rail calls it. |
| `kind` | `take` (the default) or `hold`. |
| `summary` | What happens and what it proves. Shown under the scene while it is selected. |
| `startState` | The screen this take starts from. Shown with the summary, and the thing most likely to be wrong when a take fails. |
| `seconds` | The budget. The rail totals them into what the demo will run to. |
| `prompt` | Required on a take, and refused on a hold. One imperative sentence per line. |
| `freezeOn` | What a hold is frozen on. |

A sheet is checked before the window opens: a take with no prompt, a hold that
carries one, a sheet with no takes in it, or a version this app does not read is
refused with a reason. Which is the point of a file — those are found before
somebody has spent a run finding out.

Holds are not listed in the app. They are made in the editor after the last
take, so the window lists only what it can record, and says what is left once
every take is in.

## Handing it over

```sh
open -a Corgy /tmp/how-corgy-works.json
```

Whether or not Corgy is already running, and never `corgy --demo` from your own
shell: macOS attaches Screen Recording and Accessibility to whatever *launched*
the process, so a window opened from an agent's terminal holds the terminal's
grants rather than the app's. `open` makes Corgy responsible for itself.

Then stop. The window is the handover: the user picks a take, reads the prompt
in the box, and presses Record — which is the one thing that has to stay theirs,
because a run drives their screen and writes to whatever the flow touches.

## What the app does with it

- The first take stands on its own and creates the video.
- Every later take is aimed into that video, at the second the last one ended —
  read back out of each run's report rather than from the `seconds` in the
  sheet, since a take never runs for exactly its budget.
- A take that never reached a video does not count as recorded, so a run
  somebody stopped part-way leaves the scene where it was.
- Once every take is in, the window says what is left in the editor: the holds,
  then the narration, in that order.
