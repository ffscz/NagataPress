# Connecting an AI tool to the NagataPress MCP endpoint

Use this when the user wants their assistant to translate the site but the tools are not connected yet, or when a connection fails. The site's own card (NagataPress > AI Provider > Translate with your own assistant (MCP)) shows the same steps with the site's endpoint filled in and a copy button; point the user there when they are in the admin.

## Contents

1. Before connecting
2. Three ways in
3. Steps per tool
4. When the connection fails
5. OAuth details for unusual clients

## 1. Before connecting

| Needed | Why |
|---|---|
| An active NagataPress license | The batch tools refuse new AI work without one (the status tool still answers). |
| At least one target language | Languages > Target Languages. The tools translate only into configured languages. |
| Enable the MCP endpoint switched on | NagataPress > AI Provider, MCP card, step 1. Off by default (the feature is marked experimental). Not needed for the connector route. |
| HTTPS | Application Passwords and OAuth both need it. Only a client on the same machine can connect without it. |
| Pretty permalinks | OAuth needs them (Settings > Permalinks, anything but Plain). |
| A user allowed to translate | Capability `nagatapress_translate` (administrators get it on activation or update) or `manage_options`. The setup tools (languages, style, Never Translate, glossary, visitor switch, test cart) are listed and work for administrators only. |
| WordPress 6.9 or newer | Only for the WordPress Abilities route through another connector. |

The endpoint is `https://<site>/wp-json/nagatapress/v1/mcp` (the card prints the exact address; sites with a different REST prefix differ).

Tools that connect from their provider's cloud need the site reachable from the internet with `/wp-json/` let through unchanged by firewalls and bot protection: Claude.ai and the connectors of Claude Desktop (Anthropic's addresses are 160.79.104.0/21), ChatGPT.com and Grok.com. Tools that run on the user's machine (Claude Code, Codex CLI, Cursor, VS Code, Gemini CLI and most editors) connect from there, so a staging site behind a VPN can work for them.

## 2. Three ways in

| Way | What it means | Choose it when |
|---|---|---|
| Sign in with your WordPress account (OAuth), recommended | The tool opens the site in a browser; the user signs in as an administrator and clicks Allow on "Connect an app to NagataPress". Nothing is copied by hand, and the app reaches the NagataPress endpoint only: it may read the untranslated texts and store translations (scope `nagatapress:translate`) and, when an administrator allows it, also change the NagataPress settings and replace hand-edited translations when asked (scope `nagatapress:settings`); the consent screen lists both. Access tokens last one hour and renew with a refresh token valid for 30 days. Allow sign-in with OAuth is on by default, but works only while the endpoint is on; switching the endpoint off disconnects every app. | The tool supports OAuth for remote MCP servers (most do). |
| Application Password | Generate an Application Password on the MCP card (WordPress shows it once). The card writes it into the snippet as an `Authorization: Basic <base64 of user:password>` header. If the user already has one, I already have one fills the snippet in the browser only. It opens the whole REST API of that account, so prefer OAuth where possible. | The tool has no OAuth for remote servers (Windsurf, Cline, Roo Code, Antigravity), or OAuth is blocked. |
| A connector already on this site | Novamira, the WordPress MCP Adapter or a similar connector lists the NagataPress tools as WordPress Abilities (`nagatapress/status`, `nagatapress/get-batch`, `nagatapress/submit-batch`, `nagatapress/get-translations`, `nagatapress/fix-translations`, and for an administrator's connector the setup abilities such as `nagatapress/get-settings`). No second endpoint is needed. | The site already uses such a connector. Connect the tool to that connector as its own screen says. |

To build the Basic header by hand: base64-encode `username:application-password` (the password may keep its spaces) and send `Authorization: Basic <result>`. Never ask the user to paste a password into the chat; tell them where to put it.

## 3. Steps per tool

`<endpoint>` stands for the site's endpoint URL. "Sign in" means: the browser opens the site, the user signs in as an administrator and allows the app. `<header>` stands for `Authorization: Basic <base64>`.

| Tool | OAuth | Application Password |
|---|---|---|
| Claude Code | `claude mcp add --transport http nagatapress <endpoint>`, then `/mcp` in Claude Code, pick nagatapress, Authenticate, sign in. | `claude mcp add --transport http nagatapress <endpoint> --header "<header>"` |
| Claude Desktop | Settings > Connectors (or claude.ai > Customize > Connectors) > Add custom connector, name NagataPress, paste the endpoint, Connect, sign in. Connects from Anthropic's cloud. The Free plan allows one custom connector. | Settings > Developer > Edit config, in `claude_desktop_config.json`: `{"mcpServers":{"nagatapress":{"command":"npx","args":["-y","mcp-remote","<endpoint>","--header","Authorization:${AUTH_HEADER}"],"env":{"AUTH_HEADER":"Basic <base64>"}}}}` (needs Node.js), restart. |
| Claude.ai | Customize (Settings) > Connectors > Add custom connector, name NagataPress, paste the endpoint, Connect, sign in. Connects from Anthropic's cloud. | Not offered. |
| Codex in ChatGPT Desktop | Settings > MCP servers > Add server, transport Streamable HTTP (the dialog starts on STDIO), name `nagatapress`, URL, save, Authenticate, sign in. | In `~/.codex/config.toml` (shared with Codex CLI and the IDE extension): `[mcp_servers.nagatapress]` / `url = "<endpoint>"` / `http_headers = { Authorization = "Basic <base64>" }`, restart. |
| Codex CLI | `codex mcp add nagatapress --url <endpoint>`, then `codex mcp login nagatapress`, sign in. On Windows, if `codex` is unknown: run `.\codex` from its folder or `npm i -g @openai/codex`. | The same `config.toml` block as above. |
| ChatGPT.com | Settings > Apps & Connectors > Advanced settings > Developer mode on; Create, name NagataPress, MCP server URL = endpoint, Authentication OAuth, create, connect, allow. Storing translations needs a plan that allows tools that write (Business, Enterprise, Edu); on other plans use Codex in ChatGPT Desktop or Codex CLI. | Not offered. |
| Cursor | `.cursor/mcp.json` (or `~/.cursor/mcp.json`): `{"mcpServers":{"nagatapress":{"url":"<endpoint>"}}}`; Cursor Settings > MCP shows Needs login, click it, sign in. | Same file with `"headers":{"Authorization":"Basic <base64>"}`. |
| VS Code | `.vscode/mcp.json`: `{"servers":{"nagatapress":{"type":"http","url":"<endpoint>"}}}` (or MCP: Add Server), start the server, sign in. | Same with `"headers":{"Authorization":"Basic <base64>"}`. |
| Gemini CLI | `~/.gemini/settings.json`: `{"mcpServers":{"nagatapress":{"httpUrl":"<endpoint>","oauth":{"enabled":true}}}}`, then `/mcp auth nagatapress`, sign in. | `{"mcpServers":{"nagatapress":{"httpUrl":"<endpoint>","headers":{"Authorization":"Basic <base64>"}}}}` |
| Antigravity (editor) | Not offered. | Settings > Customizations > Installed MCP Servers > Add MCP (View raw config edits the file directly), in `~/.gemini/config/mcp_config.json`: `{"mcpServers":{"nagatapress":{"serverUrl":"<endpoint>","headers":{"Authorization":"Basic <base64>"}}}}`, then Refresh. One project only: `.agents/mcp_config.json`. |
| Antigravity CLI | Not offered. | The same file as the editor, then `/mcp` and reload the configuration. |
| Grok.com | grok.com/connectors > New Connector > Custom, paste the URL, sign in. Connects from xAI's servers; on Business and Enterprise a team admin adds the connector in the xAI console first. | Not offered. |
| Grok CLI | `grok mcp add --transport http nagatapress <endpoint>`; first use opens the browser (or press `i` in `/mcps`). | `grok mcp add --transport http nagatapress <endpoint> --header "<header>"` |
| GitHub Copilot CLI | `copilot mcp add --transport http nagatapress <endpoint>`, then `/mcp auth nagatapress`. | `copilot mcp add --transport http --header "<header>" nagatapress <endpoint>` (kept in `~/.copilot/mcp-config.json`, available at once). |
| Roo Code | Not offered. | MCP panel > Edit Global MCP, `mcp_settings.json`: `{"mcpServers":{"nagatapress":{"type":"streamable-http","url":"<endpoint>","headers":{"Authorization":"Basic <base64>"}}}}`; one project: `.roo/mcp.json`. |
| Amazon Q | `~/.aws/amazonq/mcp.json`: `{"mcpServers":{"nagatapress":{"type":"http","url":"<endpoint>"}}}`; CLI: `/mcp`, open the printed address while the session stays open; IDE opens the browser itself (a server can also be added in the chat panel under the tools icon, plus, http). | Same with headers, but some versions read only the url for http servers; if refused, use the sign-in. |
| Windsurf | Not offered. | `~/.codeium/windsurf/mcp_config.json` (Settings > Cascade > MCP Servers > Manage): `{"mcpServers":{"nagatapress":{"serverUrl":"<endpoint>","headers":{"Authorization":"Basic <base64>"}}}}` |
| Cline | Not offered. | MCP Servers > Configure (Remote Servers): `{"mcpServers":{"nagatapress":{"type":"streamableHttp","url":"<endpoint>","headers":{"Authorization":"Basic <base64>"}}}}` |
| Zed | Settings > AI > MCP Servers > Add Remote Server, or `~/.config/zed/settings.json` (Windows: `%APPDATA%\Zed`): `{"context_servers":{"nagatapress":{"url":"<endpoint>"}}}`; Zed offers the sign-in itself. | Same with `"headers":{"Authorization":"Basic <base64>"}`. |
| Kilo Code | Settings > MCP > Add Server > Remote (HTTP), or `~/.config/kilo/kilo.jsonc`: `{"mcp":{"nagatapress":{"type":"remote","url":"<endpoint>","enabled":true}}}`; click the sign-in notification. | `{"mcp":{"nagatapress":{"type":"remote","url":"<endpoint>","oauth":false,"headers":{"Authorization":"Basic <base64>"}}}}` |
| OpenCode | `~/.config/opencode/opencode.json`: `{"mcp":{"nagatapress":{"type":"remote","url":"<endpoint>","enabled":true}}}`, then `opencode mcp auth nagatapress`. | Same with `"oauth":false` and the headers. |
| Other MCP client | Streamable HTTP at the endpoint; OAuth 2.1 with PKCE (S256), discovery below. | Send the header with every request. |

Client UIs change often. If a menu name does not match, the config file or command form is usually still right; the MCP card on the user's site is the most current source.

After connecting, the user gives the assistant the sentence the card shows (step 4, Then tell the assistant), for one language: "Translate this site with the NagataPress tools: call nagatapress_get_batch for lang de, translate every item as instructed, submit with nagatapress_submit_batch, and repeat until done is true." With several target languages the card names them all: "Translate this site with the NagataPress tools into all of its target languages (German (de), French (fr)), one language at a time: call nagatapress_get_batch for the language, ... then continue with the next language." With a connector: "...run nagatapress/get-batch for lang de, ... hand the translations to nagatapress/submit-batch ...". For administrators the card also offers a setup sentence (Or let it set NagataPress up with you): "Set up NagataPress on this site with me: read nagatapress_get_settings, suggest the target languages, the translation style and the terms that must never be translated (nagatapress_suggest_excluded_terms), and change only what I confirm." The same step links to the NagataPress skill.

After a plugin update the client may still hold the tool list from when it connected: if the assistant says a NagataPress tool is missing (`nagatapress_status` lists the tools the connection can call), disconnect and connect the app again. The same fixes "This connection was allowed to translate only (it was made before NagataPress 1.3.15)" for the setup tools.

## 4. When the connection fails

First ask the user to click Check the setup on the MCP card. It reports, as OK, note or problem:

| Check | Problem and fix |
|---|---|
| MCP endpoint | Switched off: turn on Enable the MCP endpoint. Only a warning, because the connector route works without it. |
| HTTPS | The site is not on HTTPS; Application Passwords and OAuth need it. |
| Pretty permalinks | Plain permalinks: OAuth cannot be offered. Settings > Permalinks, anything but Plain. |
| OAuth sign-in | Switched off: turn on Allow sign-in with OAuth, or use an Application Password. |
| WordPress Abilities for other connectors | Needs WordPress 6.9 or newer, or the switch is off. |
| REST API answers from the server itself | A cache, firewall or plugin that disables the REST API is in the way: let `/wp-json/` through unchanged. |
| Discovery at the domain root (/.well-known/) | A note only: clients fall back to the copy under the REST API. Let `/.well-known/oauth-*` reach WordPress if a client insists. |
| Authorization header reaches WordPress | Apache: keep the `.htaccess` rule that passes `HTTP_AUTHORIZATION`; nginx or a proxy: do not strip the `Authorization` header. |
| Last activity | Batches served, translations stored this way, last activity, connected apps. |

The three network checks run only while the endpoint is on, and the discovery check only while OAuth is on too.

What the endpoint's answers mean:

| Answer | Meaning |
|---|---|
| HTTP 404 `nagatapress_mcp_off` | Enable the MCP endpoint is off. |
| HTTP 401 `nagatapress_oauth_required` (with `WWW-Authenticate`) | No credentials; the client should start the OAuth sign-in. |
| HTTP 401 `rest_not_logged_in` | No credentials and OAuth is off: use an Application Password. |
| HTTP 401 `incorrect_password`, `invalid_username`, `application_passwords_disabled...` | The Application Password or user name is wrong, or Application Passwords are disabled. |
| HTTP 403 `rest_forbidden` | The account is not allowed to translate (lacks `nagatapress_translate` and `manage_options`). |
| HTTP 405 on GET | Normal: the server offers no stream. Clients must POST. |
| Tool error "AI translation is paused: ..." | The license does not allow AI work; see the status tool's `license.message`. |
| Tool error "This account is not allowed to translate the site." | The signed-in user lacks `nagatapress_translate` and `manage_options`. |
| Tool error "Changing the NagataPress settings needs an administrator account." | A setup tool, or `overwrite_manual`, called by a user who is not an administrator. |
| Tool error "This connection was allowed to translate only (it was made before NagataPress 1.3.15). ..." | Disconnect the app and connect it again while signed in as an administrator. Translating keeps working meanwhile. |

Other common causes: a security plugin or Cloudflare rule blocking `/wp-json/` or the cloud provider's addresses; a staging site behind HTTP authentication (it eats the `Authorization` header); a page cache serving cached REST answers; the user signing in with an account that is not an administrator. The Connected apps table (step 5 of the card) shows each app, user, connection time, last use and Valid until; Disconnect removes one app, which then signs in again.

## 5. OAuth details for unusual clients

- Issuer: `https://<site>/wp-json/nagatapress/v1/oauth`.
- Protected resource metadata (named in the 401 challenge): `<issuer>/protected-resource`.
- Server metadata: `<issuer>/.well-known/openid-configuration` and `<issuer>/.well-known/oauth-authorization-server`; also at the domain root in path-aware form (`/.well-known/oauth-authorization-server/wp-json/nagatapress/v1/oauth`, `/.well-known/oauth-protected-resource/wp-json/nagatapress/v1/mcp`) when the web server lets those paths reach WordPress.
- Endpoints: `<issuer>/authorize` (30 requests a minute from one address), `<issuer>/token` (authorization_code with PKCE S256, refresh_token; 60 requests a minute), `<issuer>/register` (Dynamic Client Registration, public clients only, 10 an hour), `<issuer>/revoke`.
- Clients identify themselves with a Client ID Metadata Document (an https client_id) or Dynamic Client Registration. Scopes `nagatapress:translate` and (since 1.3.15) `nagatapress:settings`; the settings scope is granted only when an administrator allows the app, and a connection made before 1.3.15 has to be connected again to get it. Access token 1 hour, refresh token 30 days, rotated at every renewal. The consent screen marks an app that registered itself (Dynamic Client Registration): its name and address are its own choice and cannot be verified, so allow it only when the user started the sign-in. Behind Cloudflare the limits count by the visitor's address from the header Cloudflare adds (filters `nagatapress_client_ip`, `nagatapress_cloudflare_ranges` for other proxies).
