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
3. Start the animation on load, finish it within the take, and hold the last
   state — the take keeps recording until its wait ends.
4. The take:

   ```json
   { "name": "Title", "startURL": "file:///tmp/<demo-name>/intro.html",
     "prompt": "Wait 4000ms" }
   ```

   The wait is the length of the card. Nothing to click, so nothing to fail.
5. Trim it in step 6 like any take: cut the page-load frames at the start.

The browser toolbar is in the recording. For a card that is fine — it reads as
part of the same browser as the rest of the demo. If it is not, keep the card
for a hold's background instead and say so to the user.

## Taste

- the product's own colours and typeface — read them from the repo's CSS or
  Tailwind config, or the live site
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
