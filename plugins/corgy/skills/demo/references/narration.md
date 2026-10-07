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

## Sound like a person

The script is one voice talking someone through a thing, not a row of captions.
Read the lines aloud in order: each should lead into the next, the way you would
say it to a colleague over your shoulder. Fragments read as a list of beats and
make the narrator sound like a slideshow.

Avoid — every line true, none of them talking to each other:

> A plain Astro project. One command: curious deploy. No hosting account, no
> cloud console, no CI, no GitHub. Published. That address is the whole setup.

Prefer — the same beats, carried by one sentence into the next:

> It's never been easier to deploy an Astro project. From your project folder,
> simply type curious deploy, and Curious takes it from there. It packs up your
> source and builds it remotely, so there's nothing to configure. A few seconds
> later, you have a live endpoint to test your project.

- Address the viewer: "you", "your project". Not "the user", not a label.
- Connect with ordinary words: "and", "so", "a few seconds later", "from there".
- One idea per line, but full sentences. A noun phrase on its own is a caption.
- Fewer, longer lines over a take beat many short ones. Silence is fine where
  nothing new happens.

### Match the voice

The user picks how the video should sound - or the project's `tone` says. The
taste rules above hold for every voice; what changes is the register.

- **Friendly** - talk to one person. Contractions, "you", an easy "so" or "and
  that's it". The Curious script above is friendly.
- **Professional** - full sentences, no slang, no exclamation marks. Say what
  it does and what that saves, plainly.
- **Energetic** - shorter sentences, active verbs, the payoff early. Still
  sentences, never a run of fragments.
- **Calm** - slower: fewer words per second (2 rather than 3), a pause after
  each step, nothing urgent.

A product's own landing-page copy is usually written to be read, not heard:
punchy fragments ("Ship faster. Zero config.") that sound like an ad when
spoken. Keep its
words and claims, but rewrite them in the chosen voice.

### Only say what is true

A "no X" list is the fastest way to say something false. Every "no account",
"no config", "no setup" has to survive the product's own docs and the recording:
if the flow asks for an email the first time, it has an account. When you cannot
check a claim, say what the viewer sees happen instead.

## Cut on sight

- "as you can see", "let's click", "we now navigate to"
- anything restating what the frame already shows
- a closing line that summarises instead of landing
- a run of fragments: "One command. No config. Published." Write the sentence
