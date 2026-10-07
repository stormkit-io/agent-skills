# Title cards and animations

Anything the product's own screens cannot show — an opening title, an end card,
a diagram that builds, a number that counts — is a small HTML page you write
and the stage records like any other take.

## The recipe

1. Write one self-contained file: `/tmp/<demo-name>/<card>.html`. Inline CSS
   and JS, no network: a font or script that loads late is a take that starts
   on a blank page.
2. Size it to the stage. The page fills the Chrome window below the toolbar,
   so design for the stage size minus about 90 points of height (1440×810 for
   the default stage), and centre everything so a few points either way do not
   matter.
3. Start the animation on a key press, not on load: the camera starts a few
   seconds after the page loads, so an animation that runs on load is over
   before the take begins. Pause everything until a keydown, with a fallback:

   ```html
   <style>body:not(.go) *, body:not(.go) *::after { animation-play-state: paused !important; }</style>
   <script>const go = () => document.body.classList.add("go"); addEventListener("keydown", go); setTimeout(go, 10000)</script>
   ```

   Finish within the take and hold the last state.
4. The take:

   ```json
   { "name": "Title", "startURL": "file:///tmp/<demo-name>/intro.html",
     "pageOnly": true, "prompt": "Key Space\nWait 4000ms" }
   ```

   Space starts the animation; the wait is its length. Nothing to click, so
   nothing to fail.
5. Insert it from just before the key press (`on_screen` says when): the
   frames before it are the card standing still.

`pageOnly` keeps the browser's tabs and address bar out, so the card fills
the frame. Record cards on a 1440×897 stage so the page is exactly 1440×810
(16:9), and design for that.

## Taste

- the product's own colours and typeface — the project's brand from
  `get_project` when it has one, otherwise the repo's CSS or Tailwind config,
  or the live site. A card loads nothing from the network, so use the brand's
  font only if it is installed on this Mac, and the closest system font if not
- the project's own logo and fonts, from the `assets` `get_project` lists.
  Download each one you use next to the card first (`curl -fsS -o
  /tmp/<demo-name>/logo.svg '<url>'` - the url lasts an hour) and refer to the
  local copy; a font file is loaded with `@font-face` from that copy
- one idea per card, readable in two seconds: the claim, or the URL to go to
- motion that explains (a request travelling, a list filling) beats motion
  that decorates
- 3 to 5 seconds. A card the narrator has to talk over for longer is a scene
  that should have been footage

## A starting point

```html
<!doctype html>
<meta charset="utf-8">
<style>
  html, body { margin: 0; height: 100%; background: #0b0d17; color: #fff;
    font: 600 64px/1.1 -apple-system, "SF Pro Display", system-ui, sans-serif; }
  body { display: grid; place-items: center; }
  h1 { margin: 0; opacity: 0; transform: translateY(24px);
    animation: in 700ms 200ms cubic-bezier(.2,.7,.2,1) forwards; }
  p { margin: 24px 0 0; font-size: 28px; font-weight: 400; color: #9aa3b5;
    opacity: 0; animation: in 700ms 700ms ease forwards; }
  @keyframes in { to { opacity: 1; transform: none; } }
</style>
<main>
  <h1>One rule. Every URL.</h1>
  <p>Dynamic pages on a static site</p>
</main>
```

## A whole video

The animated format from step 0 is one page holding the whole video: scenes
shown in turn on a timer, each one a piece of the product's landing page
rebuilt and set in motion - the headline with its highlight drawing in, a URL
typing into the input, a score counting up, issue cards sliding in, a feature
grid popping in, the end card. Same recipe as a card, longer:

- build each piece from the live page: its copy, its colours, its numbers.
  The landing page's own example (a sample report, a demo account) is fair to
  show; a figure the page does not show is not yours to make up
- one `show(scene)` timeline in the page's script, started on the key press,
  5 seconds or so a scene; a scene leaves as the next arrives
- record it as one `pageOnly` take on the 1440×897 stage. Cover the whole
  timeline with waits of 10 seconds or less (`Wait 10000ms` three times for a
  28-second page): a take can end short of one long wait, cutting off the end
  card. Check the last frame shows the end card before assembling
- no holds and no zooms: the page already paces and frames itself. Trim the
  blank frames before the key press, then narrate one line per scene
