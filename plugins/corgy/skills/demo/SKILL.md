---
name: demo
license: MIT
compatibility: Plans demos for any product. Recording one needs the Corgy macOS app (corgy.ai) 0.1.4 or newer, installed and signed in; without it the skill still produces the plan, the prompts and the narration, and the sheet it writes keeps until there is an app to open it with.
description: Plan a product demo and hand it to the Corgy macOS app. Takes the brief as an argument — `/corgy:demo "an onboarding demo for Acme"` — works out what the product is from the repo you are standing in, or from the web when the repo is not here, and comes back with a summary, a scene plan with the recording prompt under each scene, and a narration prompt. Then writes it all as a demo sheet and opens Corgy on it, where the takes are listed and each one is aimed into the video the first take made, for the user to record scene by scene. Use when the user wants a demo, an onboarding video, a release clip, a changelog GIF, or says "record a demo", "show this off", "/corgy:demo". Adapts the plan to their feedback — drop a scene, merge it into one cut, change the order — and rewrites the sheet. Never records anything itself: pressing Record stays with the person whose screen it is.
---

# Demo Planner

You have something nobody else writing this prompt has: the product itself —
its code, or failing that its docs and its live screens. That is the whole
advantage. Use it, or you will produce another cursor tour.

A good demo is **one claim, proved on screen, in under ninety seconds**.

## What you produce, in one pass

A single message containing:

1. a **summary** — what the product does, and therefore which features this demo
   should show
2. a **scene plan** — three to five scenes, or the single take the user asked
   for. Each scene is a short paragraph saying what happens and what it proves,
   **followed by its recording prompt in a fenced block**, ready to copy.
3. an **assembly sheet** — the order the takes are recorded in, where each one
   is aimed, and where the frames are held
4. a **narration prompt** — why the product exists, which the narrator writes
   the script from
5. a **demo sheet** — that plan as one JSON file, and the Corgy app opened on
   it, with the takes listed and the first one's prompt already in the box

The sheet is the handover. Everything above is for the user to read and argue
with; the file is what the app records from, so nobody pastes anything.

Nothing here blocks. Hand over the whole thing, then change it when the user
pushes back — that is step 6, and it is the normal case, not the exception.

Then **hand it over** and stop. Step 5 has the mechanics; the short version is
that you write the sheet, open the app on it, and the person at the machine
presses Record.

**Everything reaches Corgy through the Mac app or the editor.** You write a
sheet and open the app on it; the app does the rest. Never `curl`, never a
direct API call — not to drive a run, and not to read state. If you need to know
what a setting is currently set to, either the user looks, or it becomes a
precondition in the plan.

And never start a recording yourself. Not with `corgy "<prompt>"`, not with
`--demo` from your own shell, not because the last take worked. A run drives
somebody's screen and writes to whatever the flow touches, twice - and the app
only asks before performing a plan when it has a terminal to ask on, which a run
started from an agent session does not. The window is where somebody says yes.

That constraint is what makes §2 non-negotiable. Corgy's planner is looking at a
screen, not at the repo — every URL and label it does not get from you, it
guesses. You are the only one here who can read the router.

## 0. Read the argument

The argument is the brief:

```
/corgy:demo "an onboarding demo for this app"
/corgy:demo "show off the prompt history, 30 seconds, one cut"
/corgy:demo                                    # no argument: ask what the demo is for
```

It names the **subject**. Once you have one, never come back and ask the user to
pick between features — that is your job from here.

It may also settle the **shape**, and when it does, the shape is not a
suggestion:

- the length — "keep it under thirty seconds"
- the surfaces — "just the dashboard and the deploy", which is a fence around
  what the demo may show, not a starting list you are free to grow
- **the cut** — "one take", "one cut", "a single scene", "no cuts"

A shape the user gave you overrides the defaults in step 3, including the scene
count and the hold. It also survives a correction: if the plan comes back with
"no, one cut", the next plan has one scene in it, not four smaller ones. Asking
for the same shape twice is the user doing your job.

With no argument at all, ask one question — what the demo is for and who watches
it — and nothing else.

## 1. Understand the product

Before scenes, before prompts: what is this thing, and what would make someone
believe in it.

**If the repo for it is here**, read it. That is the better source and it is
cheap:

- `README.md`, `AGENTS.md` / `CLAUDE.md`, `docs/` — the product in its own words
- the router and the page components — the real surfaces, in the order a user
  meets them
- the marketing or landing page in the repo, if there is one; it already says
  what the product claims
- `git log --oneline -30` for what is recent enough to be worth showing
- the tests, which name behaviour better than the diff does

**If it is not here** — the user is asking for a demo of something you cannot
read — research it instead, and say plainly that you did:

- the product's own site and docs, via WebSearch and WebFetch
- the pricing page, which is where a product states what it thinks it is worth
- a changelog or release notes, for what is new enough to lead with

Then write down, for yourself:

- **What it does**, in one sentence, in product words
- **Who it is for**, and what they were doing before
- **The two or three moments** that would convince that person — usually a first
  success, not a feature list
- **Where it starts** — the URL or app state a viewer would begin from

If the brief names a single feature rather than the whole product, this step
shrinks: find the feature (`git log -S<symbol>`, `rg <term>`,
`gh pr list --search <term> --state all`) and demo that. One demo, one claim.
Never record a sprint.

## 2. Build the step map

Every URL, screen name, tab label and control in the prompt comes out of the
repo — or, when there is no repo, out of a page you actually fetched.
**Never invent a URL.** A path that looks obvious is the easiest thing in this
whole skill to get wrong, and nobody finds out until Corgy is already pointed at
a blank panel — by which time the user has burned a run.

In particular, a directory is not a URL segment. `pages/env/[id]/config/` may
well be mounted at `/`, and `/env/17/config` then renders nothing at all — a
page that looks half-loaded rather than throwing, which is exactly the failure
that gets misread as "still loading".

So resolve each one against the thing that actually decides it:

- the router — `routes.ts`, `routes.rb`, `App.tsx`, the file-system convention
  the framework uses; read the mounted `path`, do not infer it from the folder
- the control's own source for its visible label, so Corgy has real text to
  match — the label in the JSX, not the label you would have written
- whether a control is even rendered yet: one nested behind a toggle does not
  exist on screen until the toggle is on, and the prompt has to say so
- for a command-line step, the install or usage line from the README, verbatim

Write the map down before you write a word of prompt:

```
/teams/22/projects/917           <- routes.ts:5 (settings is path "/")
"Weekly digest"                  <- TabNotifications.tsx:137 (Switch label)
"Only when something changed"    <- TabNotifications.tsx:165, nested under it
curl -sf https://.../install.sh  <- README.md:31
```

Cite the source for each row, to yourself. A row you cannot cite is a guess, and
a guess is flagged as an open question at the end, never buried in a prompt.
When the whole map came from the web rather than from code, say so once — the
user should know which rows are read and which are inferred.

The live state matters too — a demo that flips a setting already on has no
before. You cannot read it yourself, so it goes in the plan as a precondition
the user confirms: "this has to be off before you record".

## 3. Derive the scenes

No template. Work out the scenes **this** demo needs, by answering one question:
*what does a viewer not believe yet, and what would make them believe it.* Then
cut everything that is not that.

A scene is either a **take** — something recorded — or a **hold**: a frozen
frame with nothing happening, which is how an explanation gets screen time no
action takes.

Rules, because "derived" is how a plan drifts back into a cursor tour:

- **3 to 5 scenes**, unless the user asked for a different cut. Six means you
  have two demos; two cannot carry a hold, which has to sit after an opening
  take and before the payoff.
- **One claim for the whole video.** Each scene's claim is a step toward it.
- **Every scene earns its place.** A take changes something visible; a hold has
  something on screen worth explaining. A scene that only navigates is deleted,
  and its destination becomes the next scene's start state.
- **At least one hold**, before the payoff, never at the start.
- **The problem gets a scene only if it is visible.** If the "before" cannot be
  shown, it belongs in the narration prompt, not in a scene.
- An onboarding demo may legitimately end on a sweep — several features shown in
  one take, quickly — but only as the last scene, and only once the first
  success has already landed.
- No sign-up forms, no menu tour, no settings unless the demo is about a setting.

An onboarding demo of a developer product usually has a shape like: install,
first sign-in, connect the thing, watch it work, then the sweep. Use it as a
sanity check, never as a fill-in-the-blanks.

### When the user asked for one cut

"One take", "one cut", "a single scene" means **one scene, and the plan has one
block in it.** Not four scenes recorded back to back, not four scenes you
describe as "cut together" — one take, one recording, one prompt.

Several surfaces in one cut is normal and is not a reason to split: four tabs is
four clicks in one take, not four takes. The tab switch that would have been a
cut is just the next step.

What changes in a one-cut plan:

- **no hold.** There is no frozen frame to explain over, so everything the holds
  would have carried moves into the narration prompt, spoken over the motion.
  Say so, so the user knows what they traded away.
- **the step budget is the constraint**, not the scene count. Eight steps stays
  the target and twelve is the planner's ceiling; if the surfaces the user named
  genuinely do not fit in twelve, say which one does not fit and let them cut it.
  Never resolve it by quietly splitting the take.
- **the assembly sheet shrinks** to one take, narration, export.
- **pacing carries the pauses.** A cut used to give a beat between surfaces; in
  one take that is a `wait` step, or `--cps`.

## 4. Write it out

One message, in this order.

### Summary

Two or three sentences: what the product does, who it is for, and therefore
which features this demo shows. Name the claim. If you researched it from the
web rather than reading the code, say so here in half a sentence.

### The scenes

Numbered. For each one, a short paragraph in plain product words — what happens
on screen and what it proves — then the details, then the prompt:

    ### Scene 2 — Sign in to the dashboard

    <one short paragraph: what happens, and what a viewer believes afterwards>

    Start state: <url or app state, from the step map>
    Seconds:     <budget>

    ```
    Open Google Chrome
    Navigate to https://app.example.com
    Click "Continue with GitHub"
    Wait 3000ms
    ```

A hold scene has no prompt. It gets the frame it freezes on, the seconds, and
what the narration is meant to say over it.

**Writing the prompt.** One imperative sentence per step, in order, one verb
each. Not a paragraph. The planner has seven verbs and nothing else, and what
comes back is always some sequence of them:

```
navigate   open   click   type   key   scroll   wait
```

A sentence that cannot become one of those is not an instruction, it is noise in
front of a model whose only job is emitting actions. That is how a `scroll` turns
up in a plan that needed a `click`.

What the vocabulary rules out:

- **No prohibitions.** There is no "don't" step. "Never press Enter", "do not
  open other tabs" compile to nothing, and they hand the planner *press*,
  *Enter* and *tabs* for free. The plan is a whitelist already — say only what
  happens.
- **No explanations.** "which reveals a second toggle underneath" is a step the
  planner may well try to take. The why belongs in the narration prompt.
- **No locations.** "in the middle of the panel", "at the top right", "further
  down the page" all read as `scroll` — the system prompt asks for one by name
  whenever a part of a page gets described.
- **`type` takes no target.** It goes wherever the focus already is, so the step
  before it is what puts it there: a click, or a Tab. Describing the field buys
  nothing.
- **Pacing is a flag.** "at an unhurried pace" is not in the schema. `--cps 12`
  is.

Eight steps or fewer, so the scene stays clear of the planner's twelve-step
ceiling. Each scene still needs its own start state — a later take does not
inherit the previous scene's screen — and a control that only appears after
another one still needs its opener as a step of its own.

The recorder is the Mac app, and it resolves every target through the macOS
accessibility tree: **never a CSS selector**, shortest distinctive label, and
every hand-over between applications spelled out, including the ones that go
back. `references/recorder.md` — read it before writing prompts; the rules there
fail the run, not the polish.

Good — every line is one verb, and each application change is its own step:

> Open Terminal
> Type curl -sf https://install.example.com | sh
> Key Enter
> Wait 8000ms
> Open Google Chrome
> Navigate to https://app.example.com

Bad — invented path, a location, an explanation, and a prohibition:

> Go to the environment settings page and scroll to the switch at the top right,
> which reveals a second one underneath. Do not touch anything else.

### How it goes together

The scenes are separate recordings; this is what makes them one video. Short,
because the app carries most of it now: the takes are recorded in order and each
one is aimed for you, so what is left to say is the freezes, then the narration,
then export — in that order, and the order is not advice. Both of those steps
destroy work when they are taken early: a take aimed inside a frozen frame lands
at its edge, and writing the narration again replaces every line.

`references/assembly.md` has the mechanics and the exact wording to hand over.

### The narration prompt

Prose, two to four sentences: what the product is for, the problem before it,
and the number if there is one. One prompt for the whole video — the narrator
writes the script in a single pass over the finished timeline.

Say what each held frame is for. The narrator is told only that a frame is held;
what it is held to explain has to come from you, and a benefit you leave out is
simply absent from the video.

Budget roughly **3 words per second** against the total. `references/narration.md`
has the taste rules and the lines to cut on sight.

### And then, briefly

- **what the demo claims**, and what it leaves out
- **the running time**, and the narration budget at 3 words/sec
- **before you record** — the app running and reachable, signed in, data seeded
  and readable (realistic names, no `asdf`), a clean window, any setting that has
  to start off. These go in the sheet's `preconditions` as well as in the
  message: the app carries them, and somebody recording tomorrow reads the file
  rather than this conversation.
- **what the runs will actually do.** Corgy rehearses and then records, so
  **anything a scene commits, it commits twice** — and three scenes that each
  save something save it six times. If a scene saves a setting, sends an email,
  takes a payment, or deletes something, say so plainly and name the target.
  Never hand somebody a sheet aimed at production without saying it writes there
  twice.
- **open questions** — any step-map row you could not cite

Then write the sheet and open the app on it, which is step 5.

## 5. Hand it over

Write the sheet, open the app on it, and stop.

```sh
cat > /tmp/<demo-name>.json <<'JSON'
{ …the plan… }
JSON

open -a Corgy /tmp/<demo-name>.json
```

`references/sheet.md` has the format and the fields. Two rules from it that are
easy to get wrong from here:

- **`open -a Corgy <file>`, never `corgy --demo <file>`.** macOS attaches Screen
  Recording and Accessibility to whatever *launched* the process, so a window
  opened from your shell holds that terminal's grants rather than the app's.
  `open` makes Corgy responsible for itself, and works whether or not it is
  already running.
- **Corgy 0.1.4 or newer.** An older copy launches and ignores the file, so the
  window opens empty. That is Check for Updates, not a broken sheet.

Then say what happens next, in a couple of lines: the takes are listed down the
side, the selected one's prompt is already in the box, and Record is theirs to
press. The app aims each later take into the video the first one made, so
nothing has to be typed into "Add to video" by hand.

And say what the runs will do, because the sheet is now a thing that can be run
several times without you in the room: **Corgy rehearses and then records, so
anything a scene commits, it commits twice.** Three scenes that each save
something save it six times. Name the target - a setting, an email, a payment, a
delete - in the same breath as handing the file over.

### If it does not open

- **`open` fails** — it exits non-zero and says why, which is the thing to
  report. Usually Corgy is not installed; the disk image is at get.corgy.ai.
  Do not go looking in `/Applications` to decide this: an app installed under
  `~/Applications` opens perfectly well from there, and `open` is the only thing
  that knows.
- **The window opens empty** — an older Corgy, or a sheet it refused. **A
  refusal is a dialog in Corgy, not something you can read**: opened this way
  there is no stderr for you to see, and the app has no Dock icon, so the alert
  can sit behind whatever else is on screen. Ask the user to bring Corgy to the
  front and read it. The reasons are all in `references/sheet.md` - a take with
  no prompt, a hold that carries one, a sheet with no takes in it.
- **The permission is missing** — the grant belongs to Corgy.app here, which is
  what `open` is for. If it is still refused, System Settings > Privacy &
  Security, then relaunch.

### What not to do

- **Do not record anything.** The whole point of the sheet is that the person
  whose screen it is presses the button.
- **Do not write the sheet into their repository.** It is a working file for one
  recording session; `/tmp` is where it belongs unless they ask otherwise.
- **The holds and the narration stay in the editor.** The app says what is left
  once the takes are in; `references/assembly.md` has the order and why two of
  those steps destroy work when they are taken early.

## 6. Adapt

The plan is a draft. Expect the user to change it, and change it without
re-litigating:

- **"drop scene 3"** — remove it, renumber, and fix what breaks: the next scene
  now starts from wherever scene 2 ended, so its start state and its first step
  change. Re-check the claim still lands without it.
- **"one cut"** — one scene, one prompt, holds gone into the narration. Re-read
  the one-cut rules in step 3.
- **"swap 2 and 4"** — the order is the argument, so say if the reordering costs
  the through-line, then do it.
- **"add X"** — if it pushes past five scenes or twelve steps, say what has to
  come out and pick one, rather than quietly overflowing.
- **"shorter"** — cut a scene, not the seconds off every scene. A rushed take
  reads as a broken demo.

Reprint only what changed — the affected scenes with their prompts, the order if
it moved, and the narration prompt if the claim moved. Do not reprint the whole
plan for a one-scene edit.

What happens to the sheet depends on whether anything has been recorded yet, and
getting this wrong costs somebody their morning.

**Before the first take** — write the sheet again to the same path and open it
again. The app reads the file when it opens it, so a sheet edited on disk is not
a sheet the window knows about, and somebody recording yesterday's scenes while
reading today's message is the failure this whole arrangement exists to avoid.
Say the window has been reloaded, so nobody wonders which version is on screen.

**Once a take is in the video, do not reopen it.** Opening a sheet starts a new
session: the recorded takes and the video they went into are forgotten, so the
next take creates a *second* video and the demo becomes two halves that have to
be reshot. Hand over the changed prompt for that one scene instead - they can
paste it into the box, which is what the box is - and leave the window alone.
If the change is big enough to want the whole sheet again, say plainly that it
means starting the recording over.

## Taste

- Show the outcome first if it is striking, then how it was reached
- One thread from first frame to last; the end should answer the beginning
- Silence beats filler
- Nothing on screen the claim does not need
