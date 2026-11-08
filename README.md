# More About for Cursor

Manage your [More About](https://moreabout.us) link-in-bio pages, and draft posts in Social Queue, from Cursor, Grok Bot, or any app that speaks the Model Context Protocol.

- Add, edit, reorder, and remove the links, section headers, music, and videos on your page.
- Stage a photo or a vertical video and save it as a Social Queue draft for X, Facebook Pages, or Instagram. One sign-in can own several pages, one page per business (for example Brionic Payments).
- The agent cannot approve or publish. `schedule_social_post` always creates a draft. You approve it while signed in to that page, in the dashboard: **Approve**, **Approve & schedule**, or **Post now**.

This plugin connects to the hosted More About MCP server at `https://moreabout.us/mcp`. Nothing in the plugin runs on your machine, and the plugin contains no API keys or OAuth client secrets. Sign-in is discovered from the server.

## Install

Install **More About** from the Cursor or Grok Bot marketplace, or add the server yourself:

```json
{
  "mcpServers": {
    "moreabout": {
      "type": "http",
      "url": "https://moreabout.us/mcp"
    }
  }
}
```

## Authentication

The server is Streamable HTTP. A call without a token gets `401` and a `WWW-Authenticate` header that points at `https://moreabout.us/.well-known/oauth-protected-resource`. That document names the authorization server, [Brionic ID](https://id.brionicx.com).

The first time a tool is used, the app opens Brionic ID. The connection uses the authorization-code grant with PKCE (`S256`) and can refresh the session (`offline_access`). The app registers itself as a public client, so there is no client secret to paste in. Scopes are `openid`, `profile`, `email`, `offline_access`, `mcp:read`, and `mcp:write`. Reading the page uses `mcp:read`. Changing the page or Social Queue uses `mcp:write`.

You can disconnect the app any time under Brionic ID → Connected apps. Access stops within a minute of being revoked.

Prefer a key? In More About go to **Account → AI agents**, create a personal API key, and send it as `Authorization: Bearer ma_live_…`. Do not put that key in this plugin.

## Tools

| Tool | What it does |
| --- | --- |
| `list_pages` | Every page this sign-in owns, with each page's connected socials |
| `get_my_profile` | Plan, page URL, Social Queue slots, timezone |
| `list_links` | Every block on your page, top to bottom |
| `upsert_link` | Add or edit a link, header, music, or video block |
| `reorder_links` | Set the order of blocks |
| `delete_link` | Remove a block (restorable from the dashboard) |
| `list_connected_socials` | Which social accounts are connected on a page, and what they accept. Takes `page` |
| `upload_media` | Stage a photo or video from a public URL or a one-time upload link. Takes `page` |
| `list_social_queue` | Drafts, scheduled, published, failed, and canceled posts. Takes `page` |
| `schedule_social_post` | Always creates a draft. Has no `mode`. Takes `page`. Refuses `mode`, `status`, `approve`, `publish`, and `owner_approved` |
| `update_social_post` | Edit a draft's caption, platforms, media, or time. Cannot approve or publish. Takes `page` |
| `cancel_social_post` | Delete a draft or cancel a scheduled post. Takes `page` |

`list_pages`, `get_my_profile`, `list_links`, `list_connected_socials`, and `list_social_queue` are read-only. `delete_link` and `cancel_social_post` are destructive. The server marks the read-only tools with `readOnlyHint`, and marks `delete_link` and `cancel_social_post` with `destructiveHint`. The live set of tools is whatever the signed-in server returns from `tools/list`. This table is the set this plugin documents.

`page` is the page handle or id. Every Social Queue tool takes it. It is required when the account owns more than one page.

## Social Queue

Social Queue is a Gold and Diamond feature. Gold holds 10 queued posts and Diamond holds 20. Drafts count toward the queue. Page tools work on every plan.

Publishing is implemented for **X**, **Facebook Pages**, and **Instagram**. TikTok is disabled. LinkedIn, Telegram, and Bluesky are not implemented.

1. `list_pages` lists your pages and the socials connected to each one.
2. `list_connected_socials` with `page` lists what that page can post to.
3. `upload_media`, then `schedule_social_post` with `page`, a caption, and media, creates a draft. The tool has no `mode`. Sending `mode: "scheduled"`, `status`, `approve`, `publish`, or `owner_approved` is refused.
4. `list_social_queue` with `page` lists the queue.
5. `update_social_post` with `page` edits a draft's caption, platforms, media, or time. It cannot move a post out of draft.
6. `cancel_social_post` with `page` cancels a queued post.

There is no MCP approve or publish tool. Approval happens only in the dashboard, by the owner signed in to that page: [Social Queue](https://moreabout.us/creator/socialq?tab=calendar&view=list&status=draft), using **Approve**, **Approve & schedule**, or **Post now**. An agent stops at the draft and tells you it is waiting there. Asking it to approve, post, or schedule does not publish the post.

## Example prompts

- "List my More About pages and the socials connected to each."
- "Add a link called Portfolio to https://example.com at the top of my page, then list my links."
- "On the Brionic Payments page, stage https://moreabout.us/assets/img/icon-512.png and draft an X post for tomorrow at 9am saying hello."
- "Show Social Queue drafts for the Brionic Payments page."
- "Cancel that draft."

Approving the draft is not a prompt for the agent. Sign in to that page and use **Approve**, **Approve & schedule**, or **Post now** in Social Queue.

## Privacy, terms, and support

- Privacy policy: https://moreabout.us/privacy
- Terms of use: https://moreabout.us/terms
- FAQ: https://moreabout.us/faq
- Support: support@moreabout.us
- Feedback: https://moreabout.us/feedback

The server only acts on pages the signed-in creator owns. It does not see other creators' data or subscribers' details. Files staged for a post and never used are deleted after 24 hours.

## For reviewers

1. Add the plugin (or the `mcp.json` above) and enable the `moreabout` server.
2. Ask: "List my More About pages." The app opens Brionic ID. Create an account or sign in, then approve access that includes changes.
3. If the account has no page yet, sign in once at https://moreabout.us/signin to create one, then retry.
4. Ask: "Add a link called Portfolio to https://example.com at the top of my page, then list my links." Check the page at the URL the tools returned.
5. Ask: "Move Portfolio to the bottom" and "Remove the Portfolio link."
6. Social Queue needs a Gold or Diamond page with X, a Facebook Page, or Instagram connected. Ask: "On my page, stage https://moreabout.us/assets/img/icon-512.png and draft an Instagram post for tomorrow at 9am saying hello." The post appears as a draft at https://moreabout.us/creator/socialq?tab=calendar&view=list&status=draft. The agent should stop and say it is waiting for dashboard approval.
7. Ask: "I approve that draft. Post it now." The agent should refuse and point at the dashboard. It must not call a publish or approve tool. To publish, sign in as the owner of that page and use **Approve**, **Approve & schedule**, or **Post now**.
8. Ask: "Cancel that draft."

For a reviewer account with Social Queue enabled, email support@moreabout.us.

## License

MIT. See [LICENSE](LICENSE).
