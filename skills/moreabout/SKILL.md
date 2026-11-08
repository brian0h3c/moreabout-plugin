---
name: moreabout
description: Manage a creator's More About link-in-bio pages and Social Queue drafts for X, Facebook Pages, and Instagram. Use when the user mentions More About, moreabout.us, a link-in-bio page, page links or blocks, Social Queue, or wants to draft, list, or cancel a post. Drafts wait for the page owner to approve them in the moreabout.us dashboard. There is no MCP approve or publish tool.
---

# More About

The `moreabout` MCP server acts on pages the signed-in creator owns. Sign-in is through Brionic ID. One sign-in can own more than one page, one page per business (for example Brionic Payments).

## Start here

Call `list_pages` first. It returns every page this sign-in owns, and the social accounts connected to each page. Pass `page` (the handle or the id) on every Social Queue tool. The server requires `page` when the account owns more than one page. Pass it every time, including when there is only one page.

`get_my_profile` tells you the plan (Free, Gold, or Diamond), how many page blocks and Social Queue slots are used, and the timezone. Social Queue needs Gold or Diamond. On Free, say so and link the plan page it returns instead of trying.

## Page blocks

- `list_links` returns every block in exact top-to-bottom order. Block ids look like `link:12` or `video:3`. Always list before editing so you use current ids.
- `upsert_link` without `id` creates a block: `link` (any http/https URL), `header` (a section title, no URL), `music` (a track link), `video` (YouTube or Vimeo). With `id` it edits title, URL, visibility, or `enabled`. `position` is 1-based across the whole page.
- `reorder_links` with every id sets the exact order. With only some ids, those swap among the slots they already hold and the rest stay put. To move one block, prefer `upsert_link` with `position`.
- `delete_link` moves a block to Removed, where the creator can restore it. Confirm with the user before deleting.

## Publishing happens in the dashboard

`schedule_social_post` always creates a draft. It has no `mode`. Do not pass `mode`, `status`, `approve`, `publish`, or `owner_approved`. The server refuses those arguments.

`update_social_post` edits a draft's caption, platforms, media, or time only. It cannot move a post out of draft.

There is no MCP tool that approves or publishes a post. Approval happens only when the owner is signed in to that page on moreabout.us and uses Social Queue: **Approve**, **Approve & schedule**, or **Post now**.

After you create or edit a draft, stop. Tell the user the draft id, the page, the caption, the platforms, the media, and the time, and that it is waiting for their approval in the dashboard at https://moreabout.us/creator/socialq?tab=calendar&view=list&status=draft. Do not try to approve it. If they say "I approve", "post it now", or "schedule it" in chat, tell them those actions are the dashboard buttons and this server will not perform them.

## Posting to socials

1. `list_pages`, then choose the page. One page per business.
2. `list_connected_socials` with `page`. Only post to platforms that are both connected on that page and implemented: **X**, **Facebook Pages**, and **Instagram**. TikTok is disabled. LinkedIn, Telegram, and Bluesky are not implemented. Do not promise those, or Threads or YouTube Shorts. If the user names one, say it is not available and name the connected implemented platforms instead.
3. Stage the file with `upload_media` and the same `page`. One photo or one vertical video (up to 59 seconds) per post.
   - Public file on the web: `action: "import_url"` with an https `source_url`.
   - Local file: `action: "init_upload"`, then PUT the raw bytes to the returned `upload_url` (for example `curl -T reel.mp4 "<upload_url>"`). Never paste file contents into a tool call.
   - Check progress with `action: "status"`.
4. `schedule_social_post` with `page`, `media_ids` (exactly one), `platforms`, `caption`, and `scheduled_at` when the user has a time in mind. Do not send `mode`, `status`, `approve`, `publish`, or `owner_approved`.
5. Stop and point the user at the dashboard. The draft stays a draft until they approve it there.

Times are ISO-8601. Without an offset (`2026-11-07T09:00:00`) they are read in the creator's timezone from `get_my_profile`. Pass `timezone` to use another, or end with `Z` for UTC. The product default timezone is Pacific until the creator changes it. Times must be in the future and within a year.

Drafts and scheduled posts both take a queue slot until they publish or are canceled. If the queue is full, tell the user rather than retrying.

A draft is not a published post. Report the status the tool returns. Do not tell the user a post is live on a network unless `list_social_queue` says it was published.

## Managing the queue

- `list_social_queue` with `page` and an optional `status` (`draft`, `scheduled`, `published`, `failed`, `canceled`).
- `update_social_post` with `page` changes caption, platforms, media, or time on a draft. It does not approve or publish. Do not send `mode`, `status`, `approve`, `publish`, or `owner_approved`.
- `cancel_social_post` with `page` deletes a draft, or cancels a scheduled post and keeps it in history. Confirm first.

## Errors

Tool errors explain what to change, for example a missing `page`, an unconnected platform, a caption over a platform's limit, a full queue, or a refused approval field. Relay them plainly. Do not retry with `mode`, `status`, `approve`, `publish`, or `owner_approved`. Do not quietly drop a platform or shorten a caption without asking. If `page` is required, call `list_pages` and retry with the handle or id the user chose.
