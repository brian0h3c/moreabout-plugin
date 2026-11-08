---
name: moreabout
description: Manage a creator's More About link-in-bio page and Social Queue drafts for Instagram, Facebook, YouTube Shorts, X, LinkedIn, Telegram, Threads, and Bluesky. Use when the user mentions More About, moreabout.us, their link-in-bio page, page links or blocks, Social Queue, or wants to draft, schedule, list, or cancel a reel, Short, or photo. Posts stay drafts until the user explicitly approves publishing.
---

# More About

The `moreabout` MCP server acts on the signed-in creator's own page and Social Queue. Sign-in is through Brionic ID. Every tool only ever sees that one creator's data.

## Start here

Call `get_my_profile` first. It tells you the handle, plan (Free, Gold, or Diamond), how many page blocks and Social Queue slots are used, the creator's timezone, and which platforms are connected. Social Queue needs Gold or Diamond. On Free, say so and link the plan page it returns instead of trying.

## Page blocks

- `list_links` returns every block in exact top-to-bottom order. Block ids look like `link:12` or `video:3`. Always list before editing so you use current ids.
- `upsert_link` without `id` creates a block: `link` (any http/https URL), `header` (a section title, no URL), `music` (a track link), `video` (YouTube or Vimeo). With `id` it edits title, URL, visibility, or `enabled`. `position` is 1-based across the whole page.
- `reorder_links` with every id sets the exact order. With only some ids, those swap among the slots they already hold and the rest stay put. To move one block, prefer `upsert_link` with `position`.
- `delete_link` moves a block to Removed, where the creator can restore it. Confirm with the user before deleting.

## Publishing is approval-gated

Any post the user has not explicitly approved stays a draft. Never publish a post, and never move one out of `draft`, without that approval.

Explicit approval is a clear yes to let that specific post go out, after the caption, platforms, media, and time are settled. Examples: "approve it", "yes, publish that draft", "I approve posting this caption to Instagram at 9am".

These are not approval: "schedule this", "queue it", "post this tomorrow", "share it to Instagram", "draft it", or any request that only describes the post. Asking to schedule means create a draft for that time, then stop and ask.

If one message both states the exact caption, platforms, and time and explicitly approves publishing that post, create the draft first, then approve it with `update_social_post`. If you filled in any detail the user did not state, stop at the draft and ask.

## Posting to socials

1. `list_connected_socials` to see which platforms are connected and what each accepts. Only post to connected platforms. The product documents Instagram, Facebook, YouTube Shorts, X, LinkedIn, Telegram, Threads, and Bluesky. YouTube Shorts takes videos only. Ignore a platform the account has not connected, even if the user named it, and say so.
2. Stage the file with `upload_media`. One photo or one vertical video (up to 59 seconds) per post.
   - Public file on the web: `action: "import_url"` with an https `source_url`.
   - Local file: `action: "init_upload"`, then PUT the raw bytes to the returned `upload_url` (for example `curl -T reel.mp4 "<upload_url>"`). Never paste file contents into a tool call.
   - Check progress with `action: "status"`.
3. `schedule_social_post` with `mode: "draft"`, `media_ids` (exactly one), `platforms`, `caption`, and `scheduled_at` when the user has a time in mind. Always pass `mode: "draft"`. Do not pass `mode: "scheduled"` here.
4. Tell the user the draft id, caption, platforms, media, and time. Stop there unless this conversation already contains explicit approval of those exact details.
5. Only after that approval, call `update_social_post` with `mode: "scheduled"` to let it go out at the draft's time. `mode: "draft"` holds a scheduled post back.

Times are ISO-8601. Without an offset (`2026-11-07T09:00:00`) they are read in the creator's timezone from `get_my_profile`. Pass `timezone` to use another, or end with `Z` for UTC. The product default timezone is Pacific until the creator changes it. Times must be in the future and within a year.

Drafts and scheduled posts both take a queue slot until they publish or are canceled. If the queue is full, tell the user rather than retrying.

Social Queue is in beta. A draft, and even an approved scheduled post, is not proof that the platform has published it. Automatic publishing is still being turned on per platform. Report the status the tool returns. Do not tell the user a post is live on a network unless `list_social_queue` says it was published.

## Managing the queue

- `list_social_queue` with an optional `status` (`draft`, `scheduled`, `published`, `failed`, `canceled`).
- `update_social_post` changes caption, platforms, media, time, or mode. Changing the content of a draft does not approve it. `mode: "scheduled"` is the approval step and requires explicit approval of the post you are about to release. `mode: "draft"` holds a scheduled post back.
- `cancel_social_post` deletes a draft, or cancels a scheduled post and keeps it in history. Confirm first.

## Errors

Tool errors explain what to change, for example an unconnected platform, a caption over a platform's limit, or a full queue. Relay them plainly. Do not quietly drop a platform or shorten a caption without asking.
