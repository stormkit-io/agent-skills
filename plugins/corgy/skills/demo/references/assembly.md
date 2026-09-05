# Assembly

How scenes become one video. The order is not advice — two of these steps
destroy work if they are taken early.

Everything here happens in the Mac app or in the editor. Nothing here is an API
call, and nothing here is something an agent does on somebody's behalf.

## 1. Record the takes, in order

From the window the sheet is open in. Pick a take, read the prompt in the box,
press Record.

The first one creates the video. Every later one is aimed into it for you: the
app reads the video and the moment back out of each run's report and fills in
"Add to video" and "at" before you get there, so the takes land end to end
without anybody typing an id.

Which is worth knowing when it looks wrong. The number in "at" is where the last
take actually *landed*, not where it was pointed - a take aimed inside a frozen
frame, or into a sliver too close to an existing cut to leave a clip either
side, snaps silently to the nearer edge. That is also the reason the freezes
come after every take rather than before.

A take is its own run from its own start state. It does not inherit the previous
scene's screen, which is why every scene in the sheet carries one.

A run stopped part-way uploads nothing, so the scene stays unrecorded and the
aim does not move. Record it again when the screen is back where the scene
starts.

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
