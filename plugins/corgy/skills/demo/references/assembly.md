# Assembly

How scenes become one video. The order is not advice — two of these steps
destroy work if they are taken early.

Everything here happens in the Mac app or in the editor. Nothing here is an API
call.

## 1. Record scene 1

In the Mac app. It creates the video. Its URL is `/videos/<id>`, and that id is
what every later step needs — the assembly sheet should say so, since nothing
else can know it in advance.

## 2. Record each later take into it

A take is aimed at a moment, and the moment is the playhead:

- open the video and put the playhead where the scene goes
- run `corgy "<that scene's prompt>" --into <id> --at <s>`, or type the id and
  the time into the app window's own "Add to video" and "at" fields, which is
  the path when `corgy` is not on the PATH

The Add panel shows that command line ready to copy, with the id and the current
playhead already filled in.

The take lands at the moment it was aimed at; a landing inside an existing clip
splits it. Aim again if the playhead has moved since — the editor says which
moment it actually holds.

Except where nothing can be cut. A point inside a frozen frame, or a sliver too
close to an existing cut to leave a clip either side, snaps to the nearer edge
of it instead — silently, and somewhere other than the playhead showed. Which is
the reason the freezes come after every take, not before.

A take is its own run from its own start state. It does not inherit the previous
scene's screen.

## 3. Add the holds

A hold is a frozen frame, and it is how an explanation gets screen time that no
action takes. In the Clip panel: put the playhead on the frame, set the seconds,
press "Freeze the frame".

Freeze on something worth looking at. A held frame of an empty page explains
nothing, and the narrator is told only that a frame is held — what it is held
*for* has to come from the narration prompt.

## 4. Write the narration — last

Voice panel → "What this demo shows" → paste the narration prompt → "Write the
narration". On a demo that already has a script the button reads "Write it
again", which is the case anyone re-running this step is in.

Last, for two reasons:

- the script is written **over the whole timeline in one pass**, so a take or a
  freeze added afterwards is not in the version the narrator saw
- writing it again **replaces every line**, so any line edited by hand before
  this point is gone

Which is also the rule to hand the user: do the hand-edits after this step, never
before it.

## 5. Export

Then the look, the music and the voice if they want them — none of it changes
what the demo claims.

## What the sheet has to carry

For each scene, in order: where the playhead goes and the prompt to paste. Then
the freezes, then the narration prompt, then export.

And the warning, per scene: Corgy rehearses before it records, so **anything a
scene commits, it commits twice**. Three scenes that each save a setting save it
six times. Name the target.
