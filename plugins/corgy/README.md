# Corgy

[Corgy](https://www.corgy.ai) turns screen recordings into narrated product
demos. This plugin lets Claude Code plan and make those demos from your
codebase.

| Command | What it does |
| --- | --- |
| `/corgy:make` | Makes the whole video. Plans the scenes from your repo, records every take in a clean Chrome window through the Corgy Mac app, checks each take, cuts and assembles it, adds zooms, music and the look, writes and voices the narration, and hands back a video to review. |
| `/corgy:demo` | Plans the demo and hands it to the Corgy Mac app as a scene sheet, for you to record scene by scene. |

## Requirements

- **macOS** with [Corgy for Mac](https://www.corgy.ai/getting-started)
  installed and signed in. Recording happens in the app.
- For `/corgy:make`, **Let agents record** turned on in the app's
  Settings → Agents. Corgy asks the first time an agent starts a recording.
- A **Corgy account**. Writing a script and generating the voiceover use
  credits; Claude asks before spending them.

## Install

```
/plugin marketplace add stormkit-io/agent-skills
/plugin install corgy@stormkit
```

The plugin connects the Corgy MCP server (`https://api.corgy.ai/api/mcp`),
which is how Claude edits the video. The first time it is used, run `/mcp` and
sign in to Corgy in the browser window that opens.

## Use

```
/corgy:make "a 60-second onboarding demo for our dashboard"
/corgy:demo "the export feature, 60 seconds, for the changelog"
```

Inside the product's repo, Claude reads the code for the real URLs, labels and
flows. Outside it, it researches the product's site and says so.

Nothing is published unless you ask: the finished video opens in the Corgy
editor for you to review, and a public share page is made only on request.

## What it runs and sends

The plugin has no hooks, scripts or local servers. Its skills tell Claude to:

- **Read your repository** to plan the demo, or the product's public website
  when you are not in its repository.
- **Write files under `/tmp`**: the demo sheet (`/tmp/<demo-name>.json`), and
  for `/corgy:make` any title cards (`/tmp/<demo-name>/*.html`) and the brand
  assets they use.
- **Run `open -a Corgy <sheet>.json`** to hand the sheet to Corgy for Mac, and
  read the progress report the app writes beside it.
- **Call the Corgy MCP server** at `https://api.corgy.ai/api/mcp`, signed in as
  you, to edit, narrate, export and, only when you ask, publish the video.
- **Run `curl`** only against links the Corgy MCP server returns: to download
  your project's logo and fonts from your Corgy account, and to upload a music
  file you choose to Corgy's storage.

Nothing is sent anywhere other than Corgy.

## Data

Recordings, scripts and voiceovers are stored in your Corgy account. Takes are
recorded on your Mac by Corgy for Mac and uploaded to your account. See the
[privacy policy](https://www.corgy.ai/privacy).

## Support

Write to support@corgy.ai or open an issue on
[stormkit-io/agent-skills](https://github.com/stormkit-io/agent-skills/issues).
