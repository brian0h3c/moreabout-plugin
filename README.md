# More About for Cursor

Manage your [More About](https://moreabout.us) link-in-bio page, and draft posts in Social Queue, from Cursor, Grok Bot, or any app that speaks the Model Context Protocol.

- Add, edit, reorder, and remove the links, section headers, music, and videos on your page.
- Stage a photo or a vertical video and save it as a Social Queue draft for the accounts you have connected. The product documents Instagram, Facebook, YouTube Shorts, X, LinkedIn, Telegram, Threads, and Bluesky.
- An agent always leaves a new post as a draft. It does not publish, and it does not mark a post scheduled, until you explicitly approve that post.

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
| `get_my_profile` | Handle, page URL, plan, Social Queue slots, connected platforms |
| `list_links` | Every block on your page, top to bottom |
| `upsert_link` | Add or edit a link, header, music, or video block |
| `reorder_links` | Set the order of blocks |
| `delete_link` | Remove a block (restorable from the dashboard) |
| `list_connected_socials` | Which social accounts are connected and what they accept |
| `upload_media` | Stage a photo or video from a public URL or a one-time upload link |
| `list_social_queue` | Drafts, scheduled, published, failed, and canceled posts |
| `schedule_social_post` | Create a post. Agents use this only with `mode: "draft"` |
| `update_social_post` | Change caption, platforms, media, or time, or approve a draft |
| `cancel_social_post` | Delete a draft or cancel a scheduled post |

`get_my_profile`, `list_links`, `list_connected_socials`, and `list_social_queue` are read-only. `delete_link` and `cancel_social_post` are destructive. The server marks the read-only tools with `readOnlyHint`, and marks `delete_link` and `cancel_social_post` with `destructiveHint`. The live set of tools is whatever the signed-in server returns from `tools/list`. This table is the set this plugin documents.

## Social Queue

Social Queue is a Gold and Diamond feature, in beta. Gold holds 10 queued posts and Diamond holds 20. Drafts count toward the queue. Page tools work on every plan.

The four scheduling steps are all covered by the tools above:

1. `list_connected_socials` lists connected platforms.
2. `upload_media`, then `schedule_social_post` with a caption, creates a post.
3. `list_social_queue` lists the queue.
4. `cancel_social_post` cancels a queued post.

Approval is a separate step. `schedule_social_post` is called with `mode: "draft"`. After you explicitly approve that draft, `update_social_post` with `mode: "scheduled"` is what lets it go out. Asking an agent to "schedule" or "post" something is not that approval. It will save a draft and ask you.

You can also approve or discard a draft in the dashboard at [Social Queue](https://moreabout.us/creator/socialq?tab=calendar&view=list&status=draft).

Automatic publishing to each platform is still being turned on. A scheduled post is the queue entry, not a promise that the network has already published it.

## Example prompts

- "Show my More About profile."
- "Add a link called Portfolio to https://example.com at the top of my page, then list my links."
- "Which social accounts are connected?"
- "Stage https://moreabout.us/assets/img/icon-512.png and draft an Instagram post for tomorrow at 9am saying hello. Leave it as a draft."
- "I approve that draft. Schedule it for that time."
- "Show my Social Queue drafts."
- "Cancel that draft."

## Privacy, terms, and support

- Privacy policy: https://moreabout.us/privacy
- Terms of use: https://moreabout.us/terms
- FAQ: https://moreabout.us/faq
- Support: support@moreabout.us
- Feedback: https://moreabout.us/feedback

The server only acts on the signed-in creator's own page and Social Queue. It does not see other creators' data or subscribers' details. Files staged for a post and never used are deleted after 24 hours.

## For reviewers

1. Add the plugin (or the `mcp.json` above) and enable the `moreabout` server.
2. Ask: "Show my More About profile." The app opens Brionic ID. Create an account or sign in, then approve access that includes changes.
3. If the account has no page yet, sign in once at https://moreabout.us/signin to create one, then retry.
4. Ask: "Add a link called Portfolio to https://example.com at the top of my page, then list my links." Check the page at the URL `get_my_profile` returned.
5. Ask: "Move Portfolio to the bottom" and "Remove the Portfolio link."
6. Social Queue needs a Gold or Diamond page with a connected social account. Ask: "Stage https://moreabout.us/assets/img/icon-512.png and draft an Instagram post for tomorrow at 9am saying hello. Leave it as a draft." The post appears as a draft at https://moreabout.us/creator/socialq?tab=calendar&view=list&status=draft and does not publish.
7. Ask: "I approve that draft. Schedule it." Only this step may mark it scheduled. Skip it if you only want to confirm the draft.
8. Ask: "Cancel that draft."

For a reviewer account with Social Queue enabled, email support@moreabout.us.

## License

MIT. See [LICENSE](LICENSE).
