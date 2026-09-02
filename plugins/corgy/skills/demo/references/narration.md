# Narration

## Where each piece comes from

| Piece | Source |
| --- | --- |
| What happened, beat by beat | the step labels Corgy records, across every take |
| Why it matters | the narration prompt you write, from the code |
| The script | the narrator, over the whole timeline at once |

The narrator is told what happened rather than asked to infer it, and it writes
the whole script in one pass so the demo can build to something. It will not
invent a benefit you did not supply.

## Written last

One prompt for the whole video, in the editor's Voice panel, **after every take
is in place and every frame is held**. Two reasons, and both cost work if
ignored:

- the script is written over the timeline as it stands, so a take added
  afterwards is not in the version the narrator saw
- writing it again replaces every line, so a line edited by hand before this
  point is gone

Hand-edits come after. `references/assembly.md` has the order.

## The prompt

Prose, not bullets. Two to four sentences: what the feature is for, the problem
before it, the number if there is one. Leave out anything the viewer cannot see
and does not need.

> Scheduled reports go out on their own now. The weekly summary used to be
> somebody's Monday morning, and the number it quoted was already two days old
> by the time anyone read it.

## Held frames

An explanation needs screen time the action does not take, which is what a hold
is for: the frame stops and the narration says what this is.

The narrator is told only that a frame is held — "a held frame, nothing
happening". What it is held *for* has to be in the prompt, or the script will
fill the silence with whatever it can see:

> The frame holds on the two switches so there is room to say the second one
> only matters for pages that never had a .md file.

A held frame of an empty page explains nothing. Freeze on the thing being
explained.

Budget **3 words per second**. One or two holds in a demo — more and it stops
being a recording.

## Cut on sight

- "as you can see", "let's click", "we now navigate to"
- anything restating what the frame already shows
- a closing line that summarises instead of landing
