# The recorder

Corgy records with the Mac app. It drives a real cursor over the whole desktop,
so a scene can be a browser, a terminal, an editor, a native app, an OS dialog,
or Corgy itself — and a demo of a terminal and a browser is one video.

Everything reaches Corgy through the app or the editor. Never `curl`, never an
API call — not to drive a run, and not to read state either.

## Reaching it

```sh
command -v corgy || ls -d /Applications/Corgy.app
```

They are the same program. `Corgy.app/Contents/MacOS/corgy` is the binary the
bundle runs, and it shows the window when it is launched with no arguments and
records a demo when it is given a prompt. So an install that answers only with
the bundle is not missing the command line — it is missing a name on the PATH.

Two ways to get one, and they both land on `~/.local/bin/corgy`:

- **the app** — Settings → Command line → Install, which is the path for anyone
  who took the disk image
- **a build of your own**, if you work on Corgy itself, which links the local
  build to the same name instead

Which means they can fight. The app's pane says when the link points at another
copy, and Install replaces it; a link left pointing at a build directory
somebody has since cleaned is a command that resolves to nothing.

Where neither answers, Corgy is not installed and there is nothing to drive.

## The grant follows the launcher

macOS attaches Screen Recording and Accessibility to the *responsible process* —
whatever launched the binary — never to the binary itself. Run from a shell,
that is the terminal, not Corgy.app, and a permission the app holds does not
help. So a run driven from an agent session needs **that terminal** granted in
System Settings, and restarted afterwards. The failure report says as much, and
names the application it means.

## Writing a prompt for it

The planner turns the prompt into actions — `navigate`, `open`, `click`, `type`,
`key`, `scroll`, `wait` — and resolves every target through the macOS
accessibility tree.

- **Never a CSS selector.** It cannot be resolved, and the step fails. Visible
  text only.
- **Shortest distinctive label.** "Save" beats "the Save button" — resolution
  favours an exact match.
- **Quote the label as the code renders it**, not as you would have written it.
- **Name the application** the scene happens in. A demo of an application starts
  by opening that application, not by navigating to a URL. `navigate` types into
  a browser's address bar, so it is only for reaching a page — reaching one from
  a terminal means opening the browser first.
- **Name every hand-over, including the ones that go back.** Focus does not
  return on its own: a flow that leaves the terminal for Chrome and then needs
  the terminal again has to say "switch back to the terminal", or the typing
  lands in Chrome. Each one is an `open` step of its own, so count them against
  the cap below.
- **For one of several identical labels**, say which by what is next to it: "the
  Get started under Pro".
- Ask for a fresh window only when the demo needs one — "in a new Chrome window".
  Reusing what is open is quieter, and a return wants the window it was using.

## What a step is judged by

A step that fires and leaves the screen identical counts as having done nothing,
and the run replans from wherever the screen actually is. That is usually right
— it is what catches a flow that stopped working at step two — but it shapes
what is worth asking for:

- **`type` is exempt.** Typing is not a screen event, so it is judged by whether
  the characters landed in the focused element.
- **`tab` is exempt**, on the same grounds: it moves a focus ring, which is too
  small for the pixel grid and absent from the accessibility tree. It passes on
  the focus having moved, and on no reading at all.
- **Everything else is judged by the screen.** `key` with Enter or Escape,
  `click`, `scroll`: if nothing moves, the step failed. So do not ask for a
  keystroke whose whole effect is invisible, and do not ask for a click on
  something already in the state you want.

## The cap

A run is planned in **under twelve steps**. Keep a scene to eight actions or
fewer and it stays clear of the ceiling with room for the planner's own
navigation. A demo that would have been fourteen steps in one run is three
scenes that each land.
