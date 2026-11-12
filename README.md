# Impulse MCP Server

The Impulse MCP server is a hosted [Model Context Protocol](https://modelcontextprotocol.io) server from [Impulse Interactive](https://www.impulse-interactive.com). It connects an AI assistant to an Impulse account so you can work with events, registration and invitation pages, event emails, guest lists, and check-in from a chat. Edits to pages and emails are made in private drafts. Publishing or sending happens only when you ask.

Official product page and docs: <https://www.impulse-interactive.com/product/mcp-server>

## About this repository

This repository contains documentation and MCP registry metadata (`server.json`) only. The server is hosted by Impulse. Its source code is not in this repository.

## Server URL

```
https://mcp.impul.co/mcp
```

Transport: Streamable HTTP.

## Authentication

The server uses OAuth. You do not need an API key, and you should not add an `Authorization` header to your client config.

What the server advertises (observed 2026-11-11):

- An unauthenticated request returns HTTP 401 with a `WWW-Authenticate: Bearer` header that points to the protected resource metadata at `https://mcp.impul.co/.well-known/oauth-protected-resource/mcp`.
- Authorization server: `https://mcp.impul.co` (metadata at `https://mcp.impul.co/.well-known/oauth-authorization-server`).
- Authorization code grant with PKCE (`S256`), plus refresh tokens.
- Dynamic client registration, and client ID metadata documents (CIMD).
- Bearer tokens are sent in the request header.
- Scopes: `events:metadata:read-write`, `pages:read-write`, `custom-domains:read-write`, `email-templates:read-write`, `guests:read-only`, `guests:read-write`, `access:scan`, `texting-broadcast:read-write`, `pictures:read-write`.

From the product page: "The connection uses OAuth with PKCE (S256) and dynamic client registration. Your Impulse password is never shared with the AI assistant."

When you add the server, your client should prompt you to sign in to Impulse and approve access. Access is limited to the features granted to the connection and to your Impulse permissions.

## Requirements

- An Impulse account. You can [sign up](https://portal.impul.co/signup). For plan availability, see the [product page](https://www.impulse-interactive.com/product/mcp-server). It currently says MCP access is included on all Impulse plans, including Free.
- An MCP client that supports remote HTTP servers with OAuth.

## Setup

In your assistant's connector or MCP settings, add a remote server with the URL `https://mcp.impul.co/mcp`, then sign in to Impulse and approve access.

The snippets below follow each client's own documentation, with only the Impulse URL inserted. Client menus and config formats change over time, so check the linked client docs if something differs.

Packaged Impulse plugins for ChatGPT, Codex, and Claude are listed as coming soon on the [product page](https://www.impulse-interactive.com/product/mcp-server). The steps below use each client's generic remote-MCP setup.

Signed-in users can also find connected-app setup for Codex, Claude, and Cursor on the AI Agent Setup page in the Impulse portal: <https://portal.impul.co/mcp>.

### Claude (claude.ai and Claude Desktop)

Client docs: [Get started with custom connectors using remote MCP](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp), [Add an unlisted connector](https://claude.com/docs/connectors/custom/add-unlisted)

For personal accounts: **Customize > Connectors > Add custom connector**, paste `https://mcp.impul.co/mcp`, then sign in to Impulse when prompted.

### Claude Code

Client docs: <https://code.claude.com/docs/en/mcp>

```sh
claude mcp add --transport http impulse https://mcp.impul.co/mcp
```

Then sign in, either from an interactive session with `/mcp` or from the shell:

```sh
claude mcp login impulse
```

If you write a JSON config by hand, include `"type": "http"` (or `"streamable-http"`) along with `"url"`. An entry with a `url` but no `type` is treated as a broken stdio entry. Do not add an `Authorization` header; that skips OAuth.

### Cursor

Add to `~/.cursor/mcp.json` (user) or `.cursor/mcp.json` (project):

```json
{
  "mcpServers": {
    "impulse": {
      "url": "https://mcp.impul.co/mcp"
    }
  }
}
```

Then complete the Impulse sign-in when Cursor prompts you.

### Codex

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.impulse]
url = "https://mcp.impul.co/mcp"
```

Then sign in:

```sh
codex mcp login impulse
```

### VS Code / GitHub Copilot

Client docs: <https://code.visualstudio.com/docs/agents/reference/mcp-configuration>

Use the portable `mcpServers` format in a workspace-root `.mcp.json`, or in your Copilot user `mcp-config.json` (`$COPILOT_HOME/mcp-config.json` or `~/.copilot/mcp-config.json`):

```json
{
  "mcpServers": {
    "impulse": {
      "type": "http",
      "url": "https://mcp.impul.co/mcp"
    }
  }
}
```

You can also use **Chat: Open Customizations > MCP Servers > Add Server**, then complete the OAuth prompt.

You do not need an OAuth client ID. Impulse supports dynamic client registration and CIMD.

Older editor-managed configs used `.vscode/mcp.json` with a top-level `servers` object and `"type": "http"`. The VS Code docs now point new Copilot configs to the `mcpServers` format above.

### ChatGPT

If your ChatGPT plan allows custom MCP connectors or apps, add `https://mcp.impul.co/mcp` as a remote MCP server and sign in to Impulse when prompted. A packaged Impulse plugin for ChatGPT is listed as coming soon on the [product page](https://www.impulse-interactive.com/product/mcp-server).

## Tools

The server exposed **100 tools** when inspected live on 2026-11-11. Tools can change; treat this list as a snapshot. Descriptions are short paraphrases of each tool's own description.

### Account and events (9)

| Tool | What it does |
| --- | --- |
| `get_account_context` | Read the current user, authorized organization, and organization capabilities. |
| `get_skill_onboarding` | Read the first-event onboarding guide. |
| `list_events` | List all accessible events in the organization, with their feature capabilities. |
| `list_editor_events` | Find events with page or email-template access, with IDs and editor capabilities. |
| `list_event_presets` | List event presets, their required page/email slots, and automation tag definitions. |
| `get_impulse_event` | Read an accessible event's ID and attendee tags. |
| `create_event` | Create an event with its starter pages, email templates, and the selected preset's registration flow. |
| `search_event_mentions` | Find accessible events for the ChatGPT mention picker. |
| `open_events_dashboard` | Open the organization's event dashboard inside ChatGPT. Does not change event data. |

### Pages, drafts, files, and fonts (35)

| Tool | What it does |
| --- | --- |
| `get_skill_page_builder` | Read the page authoring guide before creating or editing pages. |
| `get_starter_gallery` | Load starter gallery data and preview URLs for the app. |
| `show_starter_gallery` | Show up to four preset-compatible starter designs with stored screenshots. |
| `search_page_starters` | Search the published page starter library. |
| `get_page_starter` | Inspect a full published page starter before using it. |
| `app_starter_preview` | Open a page starter preview in the Impulse app picker. |
| `search_page_examples` | Search the page example library. |
| `get_page_example` | Read one complete example page document. |
| `list_event_pages` | List an event's pages with URLs, link expressions, types, and signup fields. |
| `create_page` | Create a published page from a starter in an event. |
| `duplicate_page` | Duplicate a page to a new slug in the same event. |
| `rename_page` | Change a page's public URL slug. |
| `delete_page` | Delete a page and all its versions, including drafts. |
| `create_page_draft` | Create a private draft from a page version (defaults to live). |
| `discard_page_draft` | Delete a page draft without changing live content. |
| `publish_page_draft` | Publish a page draft and remove it. |
| `get_page_document` | Read a page's working draft, live document, or a saved version. |
| `edit_page_document` | Stage an atomic batch of set/delete edits to a page. |
| `replace_page_document` | Replace and stage the whole page document. Does not publish. |
| `list_page_versions` | List saved page versions and the selected working draft. |
| `save_page_version` | Save a named checkpoint of a page. Does not publish. |
| `restore_page_version` | Replace a page draft's contents with a saved version. Does not publish. |
| `screenshot_page` | Render desktop and mobile PNG screenshots of a page. |
| `show_page_preview` | Show an interactive page preview that follows saved changes. |
| `list_page_files` | List files saved on a page. |
| `store_page_file` | Copy an uploaded file into permanent page storage. |
| `view_page_file` | Inspect a saved page file. |
| `import_page_file` | Download a file from an HTTP(S) URL into a page's file library. |
| `list_google_fonts` | List Google Fonts with filters. |
| `search_google_fonts` | Search Google Fonts by family name or design tags. |
| `preview_google_fonts` | Compare one to six Google Fonts in a PNG specimen sheet. |
| `preview_page_fonts` | Render a page's saved font files as a PNG specimen sheet. |
| `import_google_font` | Save selected Google Font files to a page's file library. |
| `create_editor_upload` | Create an upload (up to 50 MiB) for use in a page or email. |
| `finalize_editor_upload` | Validate and finalize an uploaded file. |

### Email templates and broadcasts (35)

| Tool | What it does |
| --- | --- |
| `get_skill_email_template` | Read the email template authoring guide. |
| `get_skill_broadcast_filters` | Read the broadcast audience filter guide before writing filters. |
| `search_email_examples` | Search the email starter example library. |
| `get_email_example` | Read one complete email starter example. |
| `get_email_components` | Read the email component catalog and allowed attributes. |
| `list_email_templates` | List email templates in an event or organization library. |
| `create_email_template` | Create an email template from a starter. Does not send email. |
| `duplicate_email_template` | Duplicate an email template into a new template. Does not send email. |
| `rename_email_template` | Rename an email template. Does not send email. |
| `delete_email_template` | Delete an email template and its versions and drafts. |
| `create_email_draft` | Create a private draft from an email version (defaults to live). |
| `discard_email_draft` | Delete an email draft without changing live content. |
| `publish_email_draft` | Publish an email draft and remove it. Does not send email. |
| `get_email_document` | Read an email's working draft, live source, or a saved version, with a preview URL. |
| `edit_email_document` | Stage an atomic batch of set/delete edits to an email. |
| `replace_email_document` | Validate, compile, and stage a complete email document. Does not send email. |
| `list_email_versions` | List saved email versions and the selected working draft. |
| `save_email_version` | Save a named checkpoint of an email. Does not publish. |
| `restore_email_version` | Replace an email draft's contents with a saved version. Does not publish. |
| `screenshot_email` | Screenshot an email at desktop and mobile widths. |
| `show_email_preview` | Show an email preview that follows saved changes. |
| `store_email_image` | Save an uploaded image to the email's permanent image storage. |
| `view_email_image` | Inspect a stored email image. |
| `import_email_image` | Copy an image from a page file library or email image library into an email. |
| `send_email_template_test` | Send one real test email to an address the user specifies. |
| `send_email_broadcast_test` | Send a test of an email template to 1–10 selected event attendees. |
| `list_email_broadcast_events` | Find sending events with email-template and guest-read access. |
| `list_email_broadcasts` | List email broadcasts and test broadcasts for an event. |
| `create_email_broadcast` | Send or schedule a live email template to eligible attendees, when requested. |
| `update_email_broadcast` | Update a scheduled email broadcast before it starts. |
| `delete_email_broadcast` | Delete an email broadcast and its schedule. |
| `get_email_broadcast` | Read a broadcast's template, schedule, state, and delivery counts. |
| `get_broadcast_audience_preview` | Read the status and results of an audience preview. |
| `get_email_broadcast_preview` | Read the template document and HTML preview URL for a broadcast. |
| `preview_broadcast_audience` | Start a read-only audience preview. Does not send. |

### Guests and registration (16)

| Tool | What it does |
| --- | --- |
| `create_attendee` | Register a guest in an event. |
| `get_attendee` | Read one event guest by ID. |
| `update_attendee` | Update an event registration's email, signup fields, or tags. |
| `delete_attendee` | Permanently remove a guest from one event. |
| `query_attendees` | Query an event's attendees with filters and search. |
| `list_organization_attendees` | Search organization guest profiles across events. |
| `get_organization_attendee` | Read an organization guest profile. |
| `update_organization_attendee` | Update an organization guest's contact channels, opt-out status, or signup fields. |
| `delete_organization_attendee` | Permanently delete an organization guest profile. |
| `list_attendee_export_fields` | List the available CSV columns for an event's guest export. |
| `export_attendees` | Export all event guests to a private CSV with a short-lived download URL. |
| `list_registration_links` | List published registration pages and their public URLs. |
| `download_registration_qr` | Generate a PNG or SVG QR code for a published registration page. |
| `create_attendee_upload` | Create a JPEG/PNG selfie upload for a guest. |
| `finalize_attendee_upload` | Validate and finalize a selfie upload. |
| `attach_attendee_selfie` | Attach a finalized selfie to a guest who has no profile picture. |

### Check-in (2)

| Tool | What it does |
| --- | --- |
| `preview_attendee_access` | Preview a guest's access without recording entry or exit. |
| `record_attendee_access` | Record a guest's entry or exit using the request from `preview_attendee_access`. |

### Template tags (2)

Covers email, message, and page tags, including the custom domain landing page assignment.

| Tool | What it does |
| --- | --- |
| `list_template_tags` | List an event's email, message, or page tags and their current assignments. |
| `assign_template_tag` | Assign, move, or clear a tag on an eligible template or page. |

### App session (1)

| Tool | What it does |
| --- | --- |
| `create_app_session` | Authenticate the Impulse app's direct API requests using this connection's organization and granted scopes. This is not the user OAuth sign-in. |

## Links

- Product page: <https://www.impulse-interactive.com/product/mcp-server>
- Company site: <https://www.impulse-interactive.com>
- Sign up: <https://portal.impul.co/signup>
- AI Agent Setup in the Impulse portal (sign-in required): <https://portal.impul.co/mcp>
- Privacy: <https://www.impulse-interactive.com/privacy>
- Terms: <https://www.impulse-interactive.com/tos>
- Support: [impulse@impulse-interactive.com](mailto:impulse@impulse-interactive.com)

## License

The contents of this repository are licensed under the [MIT License](LICENSE). Copyright 2026 Impulse Interactive Inc.
