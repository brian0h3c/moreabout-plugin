---
name: moreabout
description: Manage a creator's More About link-in-bio pages (one per brand on Diamond): links and blocks, profile, look, social icons, photo gallery and 24-hour stories, click stats, and photo and short-video posts in Social Queue (Instagram, Facebook, YouTube Shorts, X, Threads). Use when the user mentions More About, moreabout.us, their link-in-bio page, its links, bio, look, gallery or stories, its stats, or wants to post or schedule a reel, Short, or photo to their socials.
---

# More About

The `moreabout` MCP server acts on the signed-in creator's own page and Social Queue. Sign-in is through Brionic ID; every tool only ever sees that one creator's data.

## Start here

Call `get_my_profile` first. It tells you the handle, plan (Free, Gold or Diamond), how many page blocks and Social Queue slots are used, the creator's timezone, and which platforms are connected. Social Queue needs Gold or Diamond; on Free, say so and link the plan page it returns instead of trying.

## Several pages

One account can run several pages, for example a creator page plus one per business (Diamond: up to 10). `list_my_pages` shows them; every other tool takes an optional `page` handle and acts on the account's main page without it. When the user names a brand or business, find its handle with `list_my_pages` and pass it as `page`. `create_page` makes a new one (Diamond only); a page past what the plan covers is paused and can be read but not changed. Each page has its own connected social accounts, so check `list_connected_socials` for that page before posting.

## Page blocks

- `list_links` returns every block in exact top-to-bottom order. Block ids look like `link:12` or `video:3`. Always list before editing so you use current ids.
- `upsert_link` without `id` creates a block: `link` (any http/https URL), `header` (a section title, no URL), `music` (a track link), `video` (YouTube or Vimeo). With `id` it edits title, URL, visibility or `enabled`. `position` is 1-based across the whole page.
- `reorder_links` with every id sets the exact order. With only some ids, those swap among the slots they already hold and the rest stay put. To move one block, prefer `upsert_link` with `position`.
- `delete_link` moves a block to Removed, where the creator can restore it. Confirm with the user before deleting.

## Profile, look and icons

- `update_profile` changes the display name, bio, or profile photo (stage the photo with `upload_media`, then pass `avatar_media_id`). The handle can only be changed in the dashboard.
- `update_appearance` changes the accent color, font, header style, button shape, name style, avatar style, and background: an animated or HD video backdrop (`background`), or a staged photo (`background_image_media_id`). Pass only what should change; `get_my_profile` shows the current look.
- `list_links` also returns `social_icons`, the row of profile icons at the top of the page. `upsert_social_icon` adds or edits one (the URL must be on that platform's site), `delete_social_icon` removes one. Icons count toward the page's block limit.

## Gallery and stories

- `list_gallery` shows the permanent gallery (up to 5 photos or videos of up to 30 seconds) and live stories (vanish 24 hours after posting).
- `add_gallery_item` with `where: "gallery"` or `"story"` and a staged `media_id`. A story is public right away; confirm before posting one.
- `remove_gallery_item` deletes a `photo:N` or `story:N` and its file for good. Confirm first.

## Stats

`get_analytics` returns clicks on the page's blocks and icons (all time and per day for 30 days), top referrers, and Social Queue results (published, failed, success rate, per platform). Page views are not tracked.

## Posting to socials

1. `list_connected_socials` to see which platforms are connected and what each accepts. Only post to connected platforms. YouTube Shorts takes videos only.
2. Stage the file with `upload_media` (the same staging works for profile photos, backgrounds and the gallery). One photo or one vertical video (up to 59 seconds) per post.
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
- `publish_social_post_now` sends a draft, scheduled, failed or canceled post out within a minute. Only when the user asked for it to go out now; platforms it already reached are skipped.
- `delete_social_post` removes a post from the queue entirely (not from the platforms it reached). Confirm first.

## Errors

Tool errors explain what to change, for example an unconnected platform, a caption over a platform's limit, or a full queue. Relay them plainly; do not quietly drop a platform or shorten a caption without asking.
