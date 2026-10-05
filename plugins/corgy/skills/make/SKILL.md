---
name: make
license: MIT
compatibility: Needs the Corgy macOS app (corgy.ai) with "Let agents record" turned on in Settings → Agents, and the Corgy MCP server connected (https://api.corgy.ai/api/mcp). Without the app it stops after the plan; without the MCP server it stops after the recording.
description: Make a finished, narrated product demo from one prompt — `/corgy:make "a 60-second onboarding demo for Acme"`. Plans the scenes from the repo (or the web), records every take itself in a clean Chrome window through the Corgy Mac app, checks each take by looking at it, cuts and assembles the video through the Corgy MCP server, adds holds, zooms, the look, music and any animated title cards, writes the narration, and hands back a video ready to review in the editor. Asks before anything that costs credits (the voiceover) and before any take that writes somewhere real. Use when the user wants the whole video made for them rather than a plan to record — "make me a demo", "produce the video", "/corgy:make". For a plan the user records themselves, use /corgy:demo instead.
---

# Make a demo

The user gives you a prompt. You give them back a video they only have to
review. Everything between is yours: the plan, the recording, the edit and the
narration. The editor at app.corgy.ai is where they look at what you made and
change what they do not like — so leave it in a state worth opening, not a pile
of clips.

You drive two things:

- **the Corgy Mac app**, by writing a demo sheet and opening it. With
  `"unattended": true` it records every take on its own, in a clean Chrome
  window of a fixed size, and writes a report beside the sheet as it goes.
- **the Corgy MCP server** (`mcp__corgy__*`), which edits the video the takes
  went into: cuts, holds, zooms, narration, look, music, export.

You never call the Corgy API any other way — no `curl` to api.corgy.ai, ever.
The only `curl` you run is the one `upload_music` hands you, to a signed link.

## The loop

1. **Plan** — the demo skill's steps 0 to 3, unchanged
2. **Write the sheet** for a self-recording run
3. **Check before rolling** — what the run will touch, and whether it may
4. **Record** — open the sheet, follow the report
5. **Look at every take** and re-record what is wrong
6. **Assemble** — trim, holds, zooms, look, music
7. **Narrate** — write the lines yourself, ask before the voiceover
8. **Review the render** and hand it over

Nothing in 1 to 6 needs the user. Say what you are doing in one line at each
step so they can watch it happen, and keep going.

## 1. Plan

Read `../demo/SKILL.md` and do its steps 0 to 3 exactly as written: read the
brief, understand the product, build the step map with a cited source for every
URL and label, and derive the scenes. The rules there — one claim, three to
five scenes, prompts as one verb per line, never an invented URL — are the
difference between a demo and a cursor tour, and none of them get looser
because nobody is reading the plan first.

Two things change because the run records itself:

- **There is no rehearsal.** A take runs once. Anything a take saves, sends,
  pays or deletes, it does once — which is better than the attended flow, and
  still something to say in step 3.
- **Every take is planned blind, from the screen.** The step map is what keeps
  it honest; a label you did not read from the source is a take that clicks the
  wrong thing with nobody there to stop it.

**Find the project before planning.** Call `list_projects`. If the user named
one, or the product plainly is one of them, that is the video's project: its
voice and look are what the video starts with, so plan title cards and the
narration around them. A new video takes a project's settings only when it is
made, which is why the project goes into the sheet rather than being set later.

Do not print the whole plan and wait. Print the claim, the scenes in one line
each, the project if there is one, and the running time, then go on to step 2.
The user asked for a video.

## 2. Write the sheet

The format is the demo skill's (`../demo/references/sheet.md`) plus three
fields for a run with nobody at the machine. `references/autopilot.md` has
them in full; the short version:

```json
{
  "title": "Dynamic pages on Stormkit",
  "project": "<id from list_projects>",
  "unattended": true,
  "stage": { "width": 1440, "height": 810 },
  "scenes": [
    { "name": "The result", "startURL": "https://sample.stormkit.dev/products/6",
      "prompt": "Wait 2000ms\nScroll down 400" },
    { "kind": "hold", "name": "Read the rule", "seconds": 4, "freezeOn": "redirects.json" },
    { "name": "Any URL works", "prompt": "Click \"Products\"\nWait 2000ms\nClick \"#121\"" }
  ]
}
```

- `project` — the first take creates the video in this project, with its voice
  and look. Leave it out when the demo belongs in no project. Add
  `"project_settings": false` only if the user asked for a video that does not
  follow the project.
- `unattended: true` — the app records the takes back to back, without asking.
- `stage` — every take happens in a dedicated Chrome with its own profile, at
  this size. None of the user's tabs, extensions or sign-ins are on camera.
  Use 1440×810, so product takes come out 16:9 (2880×1620 on Retina) and
  match title cards and any 16:9 clip the user adds. Record `pageOnly` cards
  in a separate sheet with a 1440×897 stage: the page alone is then exactly
  1440×810. Every clip in a video should have the same shape.
- `startURL` on a take — loaded with the camera off before the take starts.
  The entry take always has one. A later take has one only when it cannot be
  reached by a click from where the last take ended; without it, the stage
  stays exactly where the last take left it, which is what lets a demo click
  through instead of jumping.

A take with a `startURL` does **not** navigate to it in its prompt. The app
tells the planner the page is already open; the prompt starts from there.

Write the sheet to `/tmp/<demo-name>.json`, never into the repo.

### Title cards and animations

An intro, an outro, a diagram that moves, a number counting up: build it as a
self-contained HTML page and record it like any other take. `references/animations.md`
has the recipe. In short: write `/tmp/<demo-name>/intro.html` sized to the
stage, give the take `"startURL": "file:///tmp/<demo-name>/intro.html"` and a
prompt that starts it and waits (`Key Space`, then `Wait 4000ms`). It goes
through the same pipeline as the product footage, so it gets the same look and
the same frame size.

Use them where they earn it — an opening title, an end card with the URL, a
diagram of something the product's screens cannot show. Not between every scene.

## 3. Check before rolling

Three things, in this order.

**What the run writes.** Read every take's prompt and name anything that
commits: a setting saved, an email sent, a payment, a delete, a deploy, a
message posted. If any take does, stop and ask in one short message, naming the
target and the account — "scene 3 saves the digest setting on team Corgy's
production project; go?". Read-only demos do not ask. This is the only question
this skill asks before the voiceover.

**Whether the app may record.** Run
`defaults read ai.corgy.recorder agentsMayRecord`. If it prints `1`, go on.
Otherwise Corgy 0.1.14 and newer asks the person once, when the sheet opens:
say "Corgy will ask you to allow agents to record; press Allow" and carry
on. The report says `asking` until they answer; do not open the sheet again
while it does, or the second sheet is refused. If the report then comes back `refused` because they declined, stop and
say so. An older Corgy refuses instead of asking: tell the user to turn on
**Corgy → Settings → Agents → Let agents record**, and wait. Never set it
yourself - it is the person's consent, and writing it from a shell is
exactly what it exists to stop.

**Whether the stage is signed in.** If a take needs a login, the stage browser
has its own profile and is signed out the first time. Ask the user once to use
**Settings → Agents → Open the stage browser** and sign in, then carry on. The
profile keeps it for every later run.

## 4. Record

```sh
open -a Corgy /tmp/<demo-name>.json
```

Then follow `/tmp/<demo-name>.report.json` until its `state` is `done`,
`failed` or `refused`. Poll it from a background shell with an `until` loop,
not with repeated foreground calls — a take is tens of seconds and a demo is
minutes. `references/autopilot.md` has the report's fields.

While it runs, the user's mouse and keyboard belong to the take. Say so once
when you start it.

- **`done`** — `videoID` is the video every take went into. Go on.
- **`refused`** — `error` says why: the switch is off, the sheet came from a
  download, a permission is missing, the app is signed out, or another run is
  going. Fix what it names, then open the sheet again.
- **`failed`** — `error` names the take and the reason. The takes before it
  are in `videoID`; see "Re-recording" below.

### Re-recording

Opening a sheet again starts a **new** video. So to redo or finish takes:

1. write a new sheet holding only the takes still needed (fixed prompts, and a
   `startURL` on the first of them — the stage is wherever the failure left it)
2. open it and follow its report as before
3. move its takes into the real video with `insert_recording`: for each
   recorded take, `source_video_id` is its `recordingID`, `in_ms` 0, `out_ms`
   its `durationMs`, and `at_ms` the end of the real video (its `duration_ms`
   from `get_timeline`), or the moment the take belongs at
4. leave the scratch video alone; it is the user's to delete

The usual failures, and what to change:

- **"Which website…" / the planner asked a question** — the prompt names no
  place. Give the take a `startURL`, or name the app in its first line.
- **stuck: a step did nothing** — the label is wrong or the control is not on
  screen yet. Go back to the step map, read the label from the source again,
  and add the click that reveals it.
- **the take ran but shows the wrong thing** — found in step 5, not in the
  report. Same fix: the step map was wrong somewhere.

Two attempts per take. If a take fails twice for the same reason, stop and tell
the user what it keeps doing — that is a product or environment problem, not
a prompt problem.

## 5. Look at every take

A report that says `recorded` means the steps ran, not that the take is good.
Read the timeline and look:

- `get_timeline` — the clips, their lengths, and `on_screen`: what happened
  when.
- `get_frames` at the start, the middle and the end of every take, and at
  every moment in `on_screen` that the claim depends on.

Reject and re-record a take that shows an error page, a spinner where the
result should be, a login wall, a cookie banner over the subject, or anything
private (a real customer's name, an email, a key). Say what you saw.

## 6. Assemble

All through the MCP server. `references/editing.md` has the clocks and the
order; the rules that bite:

- **Re-read `get_timeline` after every edit.** Every time in it moves when a
  clip is cut, removed or held.
- **Trim before anything else.** Dead air at the start of a take (the page
  settling) and the end (the last wait) is the commonest flaw. Cut it with
  `cut` at both edges and `remove_clip` on the dead piece. Leave about half a
  second either side of the action; a demo that never rests reads as a glitch.
- **Holds next.** For each hold scene, `add_hold` on the frame it names, for
  its seconds. Find the frame with `get_frames`, never by guessing. Holds go
  after every take is in place — a take inserted inside a hold lands at its
  edge.
- **Zooms, by hand.** Stage takes are window captures, which record no click
  positions, so `auto_zoom` can only zoom on the centre of the frame — on
  cards and waits as much as clicks. Do not use it on staged takes. Place one
  or two `add_zoom`s on what the claim depends on (an error heading, a filled-in
  tag), aimed from a `get_frames` picture, and check them in the render.
  When you `update_zoom`, pass both `zoom_x_permille` and `zoom_y_permille`.
- **The look.** A video made in a project already has the project's voice and
  look; `get_timeline` shows the project under `project`. Keep them unless the
  user asked otherwise. A video in no project gets a background, padding and
  corner radius that suit the product's own colours.
- **Music**, only if the user gave you a file or the project library has a
  track: `upload_music` → the `curl` it returns → `finish_music`, then
  `update_music` to sit it well under the voice (volume around 150–250
  thousandths).

## 7. Narrate

Write the lines yourself with `add_line`. It is free, and you know what every
take is for, which the automatic writer does not. Use `write_script` only if
the user asks for it — it costs credits and replaces every line.

The demo skill's `references/narration.md` has the taste rules. On top of them:

- write the whole script first, as one passage read aloud, then split it into
  lines; a line is a sentence of that passage, never a caption. "Sound like a
  person" in `narration.md` has the before and after
- place each line with `start_ms` at the moment it is about — `on_screen` says
  when things happen
- about 3 words per second of the stretch it covers; a line that runs past the
  next action is a line to cut or a hold to add
- the hold scenes are where the explaining goes; the takes carry short lines
  or none

Then **stop and ask.** Print the script — every line with its time — and the
cost: `generate_voiceover` charges 2 credits per spoken second, so give the
estimate (`get_credits` for the balance). Generate it only when the user says
yes. If they change lines, `update_line` / `rewrite_line` first.

If the user said up front that the voiceover is approved ("and voice it"),
that is the yes — do not ask twice.

## 8. Review the render and hand it over

`start_export` with quality `low`, wait for `get_export` to say ready, then
`get_rendered_frames` at the opening, every hold, every zoom and the end. This
is the only place zooms, the look and subtitles are visible, so it is where
you catch a zoom that crops the subject or subtitles over the thing being
explained. Fix, render again, look again.

Then hand over, short:

- the editor link, `https://app.corgy.ai/videos/<videoID>`
- the claim and the running time
- what you did that they might not expect — a take re-recorded, a scene dropped
- what is waiting on them: the script approval and the voiceover, if not done
- any scratch videos left from re-recording, for them to delete

Do not export at high quality unless asked; the user will want to change
things first, and that is what the editor is for.

## When the user asks for changes

They will, by referring to what they see in the editor. Map it to a tool:

- "the zoom at 0:12 is too tight" → `get_timeline`, find it, `update_zoom`
- "drop the second scene" → `remove_clip` on its clips; narration over it is
  not removed with it, so remove or move those lines too
- "redo the pricing part" → a re-recording sheet for that take (step 4), then
  `insert_recording` at its place and `remove_clip` on the old one
- "say less" → shorten lines with `update_line`; regenerate only the changed
  ones with `generate_voiceover` and their `line_ids`
- anything you got wrong → `revert` to the `undo_version_id` the edit returned

Never re-run the whole loop for a change to one part of the video.
