# The Corgy demo skill

Turns Claude — or any agent that can read your repo — into the person who plans
your demo. You give it a brief, it works out what the product is (from the code
if you are standing in it, from the web if you are not), and comes back with a
summary, a scene plan with a ready-to-paste recording prompt under each scene,
the sheet that assembles them into one video, and the narration prompt.

Then, scene by scene and only when you say go, it records them with the Corgy
Mac app.

## Install

In Claude Code:

```
/plugin marketplace add stormkit-io/agent-skills
/plugin install corgy@stormkit
```

Or copy this directory into any agent's skills folder — it is a
[SKILL.md](https://agentskills.io) with three reference files beside it, and
nothing in it is specific to one runtime.

Then:

```
/corgy "an onboarding demo for our dashboard"
/corgy "the export feature, 60 seconds, for the changelog"
```

Inside the product's repo it reads the code — the router, the components, the
README — for the real URLs, the real labels and the real install line. Outside
it, it researches the product's own site and docs, and says so.

## Scenes, not one long run

A demo is three to five scenes, derived from what the product actually is rather
than from a template. Each one is either a **take** — recorded, aimed into the
video at a moment you choose — or a **hold**: a frozen frame with nothing
happening, which is how an explanation gets screen time no click takes.

That buys two things. A fluffed scene is re-recorded on its own instead of
redoing the whole demo, and each scene stays well under the planner's twelve-step
ceiling, so it lands.

## Then you change it

The plan is a draft, and the second half of the skill is adapting it. "Drop
scene 3" removes it and fixes the start state of the one after. "Let's do one
cut" collapses the whole thing into a single take, with the holds moved into the
narration. "Add the database integration" tells you what has to come out to make
room. It reprints only what moved.

## Recorded with the Mac app

Corgy records with the Mac app, which drives the whole desktop — a terminal, a
browser, an editor, a native app, or Corgy itself, all in one video. It resolves
targets through the macOS accessibility tree, so a CSS selector fails outright
and every hand-over back to an application has to be spelled out. Getting that
wrong fails the run, not the polish, which is why the skill writes the prompts
for it rather than leaving you to.

The `corgy` command is that same app, so the skill can run a take itself —
`corgy "…" --into <id> --at <s>` — and read the report back. It asks first,
every time, for every scene. It has to: the app's own "run this plan?" dialog
only appears when there is a terminal to answer it, and a run started by an
agent has none, so the app takes the silence as consent. The question in the
chat is the only checkpoint there is.

If `corgy` is not on your PATH, the app installs it: Settings → Command line →
Install.

Everything goes through the app or the editor. The skill never reaches for
`curl`.

## Why not just prompt Corgy yourself

Corgy's planner sees a screen; the skill sees the repo. It reads the mounted
route rather than guessing a path from a folder name, quotes the switch's label
out of the JSX, and knows a control is nested behind a toggle and won't be on
screen until that toggle is on. Those are the details a prompt written from
memory gets wrong, and you only find out once the run is burned.

## What you need before you paste

The skill tells you this at the end of each run, specific to the demo:

- the app running, reachable, and signed in
- data that reads well on screen — real names, no `asdf`
- a clean window
- any setting that has to *start* off, since a demo that flips something already
  on has no before

And the one that bites: Corgy rehearses before it records, so **anything a scene
commits, it commits twice** — three scenes that each save something save it six
times. The skill names the target so you can decide before pasting.

## The order that matters

The assembly sheet is ordered, and two steps destroy work if taken early. The
narration is written over the whole timeline in one pass, and writing it again
replaces every line — so it comes after the last take and the last freeze, and
hand-edits come after that.
