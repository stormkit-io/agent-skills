# Editing through the MCP server

## The clock

Every time the tools take or return is milliseconds into the **finished**
video: holds included, speed applied. A moment read from `get_timeline` can be
passed straight back. Two consequences:

- every edit that changes length — `cut` + `remove_clip`, `add_hold`,
  `set_clip_speed`, `insert_recording` — moves every later time. Read the
  timeline again before the next edit; never reuse a time from before it.
- edit from the **end** of the video towards the start when you have several
  trims to make. A trim late in the video moves nothing before it, so the
  times you already read stay right.

Inserting footage moves the narration, music and holds after the landing point
with it, and the answer's `shifted` says what moved.

## Order

1. **Trim** each take's dead edges: `cut` at both edges of the dead stretch,
   then `remove_clip` on it. End to start.
2. **Re-records** in place, if any: `insert_recording`, then remove the clip it
   replaces.
3. **Holds**: `add_hold` at the frame, for the hold scene's seconds.
4. **Zooms**: `auto_zoom`, then prune with `remove_zoom` / `update_zoom`.
5. **Look**: `set_project` + `get_project_look` + `set_look`, or `set_look` alone.
6. **Narration**: `add_line` per beat. Last among the edits — lines sit on
   finished-video times, and while holds do not move them, cuts before them do.
7. **Music**: `upload_music` → `curl` → `finish_music` → `update_music`.
8. **Voiceover**, once approved: `generate_voiceover`.

## Looking

- `get_frames` — the raw recording at a moment, about one picture a second.
  Fast, and enough to find a frame or check a take. No zooms or look.
- `get_rendered_frames` — the finished picture, from the latest render. Needs
  `start_export` (low is enough) and `get_export` ready first, and refuses a
  render older than the last edit.

## Undo

Every edit returns `undo_version_id`; `revert` to it puts the video back.
`list_versions` shows the history. Prefer reverting a bad edit to stacking a
second edit on top of it.

## Projects and the library

- `list_projects` — the user's projects. A video in a project can use its
  library.
- `list_library` / `insert_from_library` — saved clips (an intro, an outro) any
  video in the project can take. Check it before building a title card; the
  user may already have one.
- `save_to_library` — keep a card you built for the next demo of the same
  product.
