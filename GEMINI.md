# Impulse

The `impulse` MCP server is the hosted Impulse MCP server at `https://mcp.impul.co/mcp`. It connects to the user's Impulse account to work with events, registration and invitation pages, event emails, guest lists, and check-in.

- Sign-in uses OAuth. If the tools are unavailable, ask the user to run `/mcp auth impulse` and sign in to Impulse. Never ask for an Impulse password or API key.
- Start with `get_account_context` or `list_events` to find the organization and event before changing anything.
- Edits to pages and emails are staged in private drafts. Publish or send anything only when the user explicitly asks.
- Before writing a page or email, read the matching guide: `get_skill_page_builder`, `get_skill_email_template`, or `get_skill_broadcast_filters`.
- Access is limited to the scopes granted to the connection and the user's Impulse permissions.

Docs: <https://www.impulse-interactive.com/product/mcp-server>
