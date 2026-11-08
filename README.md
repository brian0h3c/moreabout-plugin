# More About for Cursor

Manage your [More About](https://moreabout.us) link-in-bio pages and line up posts in Social Queue from Cursor, Grok, Claude, ChatGPT, or any app that speaks the Model Context Protocol.

- Run several brands from one account (Diamond: up to 10 pages) and tell the agent which one to work on.
- Add, edit, reorder and remove the links, section headers, music and videos on each page.
- Stage a photo or a vertical video and queue it for Instagram, Facebook, YouTube Shorts, X and Threads.
- Posts an agent creates wait as drafts until you approve them in your dashboard, unless you ask it to schedule them outright.

This plugin connects Cursor to the hosted More About MCP server at `https://moreabout.us/mcp` and adds a skill that teaches the agent how to use it well. Nothing runs on your machine.

## Install

Install **More About** from the Cursor Marketplace, or add the server yourself:

```json
{
  "mcpServers": {
    "moreabout": { "url": "https://moreabout.us/mcp" }
  }
}
```

The first time a tool is used, Cursor opens a Brionic ID sign-in. You choose whether the agent may only read your page or also make changes. You can disconnect it any time under Brionic ID → Connected apps.

Prefer a key? In More About go to **Account → AI agents**, create a personal API key, and send it as `Authorization: Bearer ma_live_…`.

## Tools

| Tool | What it does |
| --- | --- |
| `list_my_pages` | Every page the account runs, and how many the plan allows |
| `create_page` | Add a page for another brand (Diamond) |
| `get_my_profile` | Handle, page URL, plan, Social Queue slots, connected platforms |
| `list_links` | Every block on your page, top to bottom |
| `upsert_link` | Add or edit a link, header, music or video block |
| `reorder_links` | Set the order of blocks |
| `delete_link` | Remove a block (restorable from the dashboard) |
| `list_connected_socials` | Which social accounts are connected and what they accept |
| `upload_media` | Stage a photo or video from a public URL or a one-time upload link |
| `list_social_queue` | Drafts, scheduled, published, failed and canceled posts |
| `schedule_social_post` | Queue a post as a draft (default) or scheduled |
| `update_social_post` | Change caption, platforms, media or time, or approve a draft |
| `cancel_social_post` | Delete a draft or cancel a scheduled post |

Every tool except `list_my_pages` and `create_page` takes an optional `page` handle; without it, it acts on the account's main page. Read-only tools are marked `readOnlyHint`; `delete_link` and `cancel_social_post` are marked `destructiveHint`.

## Plans

Page tools work on every plan. Social Queue is part of Gold (10 queued posts) and Diamond (20 per page). Diamond also runs up to 10 pages from one account. Drafts count toward the queue.

## Privacy and terms

- Privacy policy: https://moreabout.us/privacy
- Terms of service: https://moreabout.us/terms

The server only acts on the signed-in account's own pages. Brionic ID sign-ins are checked on every request and stop working within a minute of being revoked.

## For reviewers

1. Add the plugin (or the `mcp.json` above) in Cursor and enable the `moreabout` server.
2. Ask the agent: "Show my More About profile." Cursor opens Brionic ID; create an account or sign in, then approve access with changes allowed.
3. If the account has no page yet, sign in once at https://moreabout.us/signin to create one, then retry.
4. Ask: "Add a link called Portfolio to https://example.com at the top of my page, then list my links." Check the page at the URL `get_my_profile` returned.
5. Ask: "Move Portfolio to the bottom" and "Remove the Portfolio link."
6. Social Queue needs a Gold or Diamond page with a connected social account. Ask: "Stage https://moreabout.us/assets/img/icon-512.png and draft an Instagram post for tomorrow at 9am saying hello." The post appears as a draft at https://moreabout.us/creator/socialq?tab=calendar&view=list&status=draft and does not publish until approved there.
7. Ask: "Cancel that draft."
8. On a Diamond account, ask: "Create a page called Test Cafe at handle testcafe-review, add a Menu link to it, then list my pages." Remove it afterwards under Creator → Pages.

For a reviewer account with Social Queue enabled, email support@moreabout.us.

## License

MIT, see [LICENSE](LICENSE).
