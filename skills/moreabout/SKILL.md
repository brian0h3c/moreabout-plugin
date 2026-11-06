---
name: moreabout
description: Manage a creator's More About link-in-bio page and stage or schedule photo and short-video posts in Social Queue (Instagram, Facebook, YouTube Shorts, X, Threads). Use when the user mentions More About, moreabout.us, their link-in-bio page, page links or blocks, or wants to post or schedule a reel, Short, or photo to their socials.
---

# More About

The `moreabout` MCP server acts on the signed-in creator's own page and Social Queue. Sign-in is through Brionic ID; every tool only ever sees that one creator's data.

## Start here

Call `get_my_profile` first. It tells you the handle, plan (Free, Gold or Diamond), how many page blocks and Social Queue slots are used, the creator's timezone, and which platforms are connected. Social Queue needs Gold or Diamond; on Free, say so and link the plan page it returns instead of trying.

## Page blocks

- `list_links` returns every block in exact top-to-bottom order. Block ids look like `link:12` or `video:3`. Always list before editing so you use current ids.
- `upsert_link` without `id` creates a block: `link` (any http/https URL), `header` (a section title, no URL), `music` (a track link), `video` (YouTube or Vimeo). With `id` it edits title, URL, visibility or `enabled`. `position` is 1-based across the whole page.
- `reorder_links` with every id sets the exact order. With only some ids, those swap among the slots they already hold and the rest stay put. To move one block, prefer `upsert_link` with `position`.
- `delete_link` moves a block to Removed, where the creator can restore it. Confirm with the user before deleting.

## Posting to socials

1. `list_connected_socials` to see which platforms are connected and what each accepts. Only post to connected platforms. YouTube Shorts takes videos only.
2. Stage the file with `upload_media`. One photo or one vertical video (up to 59 seconds) per post.
   - Public file on the web: `action: "import_url"` with an https `source_url`.
   - Local file: `action: "init_upload"`, then PUT the raw bytes to the returned `upload_url` (for example `curl -T reel.mp4 "<upload_url>"`). Never paste file contents into a tool call.
   - Check progress with `action: "status"`.
3. `schedule_social_post` with `media_ids` (exactly one), `platforms`, `caption`, and `scheduled_at`.

**Default to `mode: "draft"`.** A draft waits in the creator's dashboard for them to approve with one tap. Use `mode: "scheduled"` only when the user has clearly asked for the post to go out automatically at that time.

Times are ISO-8601. Without an offset (`2026-11-07T09:00:00`) they are read in the creator's timezone from `get_my_profile`; pass `timezone` to use another, or end with `Z` for UTC. Times must be in the future and within a year.

Drafts and scheduled posts both take a queue slot until they publish or are canceled. If the queue is full, tell the user rather than retrying.

## Managing the queue

- `list_social_queue` with an optional `status` (`draft`, `scheduled`, `published`, `failed`, `canceled`).
- `update_social_post` changes caption, platforms, media, time, or mode. `mode: "scheduled"` approves a draft (only when the user asked); `mode: "draft"` holds a scheduled post back.
- `cancel_social_post` deletes a draft, or cancels a scheduled post and keeps it in history. Confirm first.

## Errors

Tool errors explain what to change, for example an unconnected platform, a caption over a platform's limit, or a full queue. Relay them plainly; do not quietly drop a platform or shorten a caption without asking.
