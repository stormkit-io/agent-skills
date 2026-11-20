# Driving a take

From Corgy 0.1.19 you drive product takes yourself, a step at a time, looking
at the stage between steps. Corgy performs each step with the real cursor and
records it. The time you spend thinking is cut out of the take once it is in
the video, so look as often as you need to.

Check the version once, before writing the sheet:

```sh
defaults read /Applications/Corgy.app/Contents/Info CFBundleShortVersionString
```

Older than 0.1.19: write prompts as before; nothing here applies.

## In the sheet

A driven take says `"driven": true`. Its `prompt` is only what the take shows,
one sentence, for the report and the narration. It still gets a `startURL`
when it needs one.

```json
{ "name": "Create a project", "startURL": "https://app.acme.com/projects",
  "driven": true, "prompt": "Creates a project called Demo" }
```

Title cards and animations stay as they are (`Key Space`, `Wait 4000ms`):
there is nothing to look at while they play.

## The exchange

Each driven take has a folder beside the sheet, `/tmp/<demo-name>.drive/`,
emptied when the take starts. When the report shows the take `recording`,
wait for the first answer:

```sh
D=/tmp/<demo-name>.drive; R=/tmp/<demo-name>.report.json
until [ -f $D/result-0.json ] || ! grep -q '"recording"' $R; do sleep 0.5; done
cat $D/result-0.json 2>/dev/null || cat $R
```

Then send one command at a time. Write it to a hidden file and move it into
place, so Corgy never reads half a command, and wait for its answer. Ids
count up from 1:

```sh
D=/tmp/<demo-name>.drive; R=/tmp/<demo-name>.report.json
printf '%s' '{"id":1,"step":{"action":"click","target":"New project","role":"button"}}' > $D/.next \
  && mv $D/.next $D/next.json
until [ -f $D/result-1.json ] || ! grep -q '"recording"' $R; do sleep 0.2; done
cat $D/result-1.json 2>/dev/null || cat $R
```

Every wait also stops when the report no longer says `recording`: the take
ended without an answer, and the report says why.

Commands:

- `{"id":n,"step":{...}}` — one step, in the planner's shape:
  - `{"action":"click","target":"<label>","role":"<optional kind>","near":"<optional text>","nth":<optional n>}`
  - `{"action":"type","text":"Demo"}` — into whatever has focus; click the field first
  - `{"action":"key","key":"Enter"}`
  - `{"action":"scroll","target":"Pricing"}` or `{"action":"scroll","dy":400}`
  - `{"action":"navigate","url":"https://..."}`, `{"action":"back"}`
  - `{"action":"wait","ms":1500}` — a beat the viewer should see; it is not cut
- `{"id":n,"look":true}` — a fresh look, nothing performed
- `{"id":n,"end":true}` — the scene is done; keep the take
- `{"id":n,"abort":true}` — throw the take away. **No answer comes**: the take
  fails, and the report shows it. Go back to following the report. A command
  Corgy cannot read ends the take the same way.

Every answer has `outcome` and, except `end`, a `look`:

- `started` (id 0), `done`, `unchanged` (the step ran and nothing on screen
  changed), `failed` (with `error`; the take is still recording), `looked`,
  `ended`
- `look.url` — where the page is
- `look.controls` — what is in view to click, as `role` and `label`: use the
  label as `target` exactly as written
- `look.screenshot` — a PNG in the folder. Read it when the controls are not
  enough to know what is on screen: after a navigation, a submit, anything
  that loads

No command for 2 minutes ends the take and keeps it, with a warning in the
report saying where it stopped.

## How to drive

- **Every step is on camera.** Thinking is cut; a wrong click is not. When
  unsure what a step will do, look first.
- **Target what the look lists**, with its role when two controls share a
  label. A `failed` step did nothing: read the error and the controls, and try
  the right label. Do not guess around the page.
- **A click that went somewhere wrong** — abort the take and record it again,
  rather than walking back on camera.
- **A sign-in page** where the product should be — abort, ask the user to sign
  in through **Corgy → Settings → Agents → Open the stage browser**, wait for
  their answer, then record the takes still needed (see "Re-recording" in the
  skill). Never type a password or an email into a take.
- **End on the result.** When the scene's last screen is up, `wait` about
  1000 ms so it lands, then `end`.
- **The check before rolling still holds.** A step that saves, sends, pays,
  deletes or deploys is one the user agreed to in step 3, or one you do not
  take.

After `end`, the next take starts: follow the report until it says that take
is `recording`, then wait for its `result-0.json` as before.
