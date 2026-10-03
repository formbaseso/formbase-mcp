# formbase MCP server

<img src="logo.png" alt="formbase logo" width="96" align="right">

The hosted MCP server for [formbase](https://formbase.so). Collect verified customer information with one request.

Your agent sends a **request** to one customer, the **recipient**: a form in your brand and their language, prefilled with what you know, so they only confirm or correct it. They upload files, sign, book or pay on the same page, without an account. The answers come back under your **field keys**, with your external ID and, when the form has a decision question, an approve, decline or changes outcome. Your agent reads them with `request_get`, or formbase calls your callback URL the moment the recipient submits.

The server lives at:

```text
https://api.formbase.so/api/mcp
```

This repository holds documentation, example configs and a Grok Build plugin manifest. There is nothing to install or run.

- [What an agent can do](#what-an-agent-can-do)
- [Connect your AI tool](#connect-your-ai-tool)
- [Try it](#try-it)
- [Tools](#tools)
- [Limits](#limits)
- [Protocol details](#protocol-details)
- [Docs](#docs)
- [Report a problem](#report-a-problem)

## What an agent can do

- **Send requests.** Ask one named person for information, with answers filled in or locked, and read what they answered.
- **Build forms.** Create a form and add, edit or remove its questions, pages and logic.
- **Publish and share.** Publish a form, create share links, translate it, and set its theme and settings.
- **Read results.** List a form's submissions and read its analytics.

Most AI tools sign in to formbase with OAuth: you paste the URL, sign in to formbase in the browser, and pick a workspace. Scripts and headless agents send an API token instead.

## Connect your AI tool

Leave any client ID, secret or token field empty. The AI tool finds the formbase sign-in page from the server and registers itself.

### Claude Code

```bash
claude mcp add --transport http formbase https://api.formbase.so/api/mcp
```

Then type `/mcp` in Claude Code to sign in.

To share the server with everyone who works on a project, commit a [`.mcp.json`](.mcp.json) to the project instead:

```json
{
  "mcpServers": {
    "formbase": {
      "type": "http",
      "url": "https://api.formbase.so/api/mcp"
    }
  }
}
```

### Claude desktop and claude.ai

Open **Customize** › **Connectors** › **+** › **Add custom connector** and paste the URL. A connector added on claude.ai also works in the desktop and mobile apps.

On Team and Enterprise, an owner adds the connector first under **Organization settings** › **Connectors**. Members then click **Connect** on it. On Free you can add one custom connector. Claude's own guide: [Get started with custom connectors](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp).

### ChatGPT

Turn on developer mode under **Settings** › **Apps** › **Advanced settings**. Then open **Settings** › **Apps** › **Create**, paste the URL, choose OAuth, click **Scan Tools**, sign in to formbase, and click **Create**.

Developer mode works on the web only. On Pro, ChatGPT can only read: tools that change something, such as sending a request or editing a form, need Business, Enterprise or Edu. On Business, only admins and owners can use developer mode. OpenAI's own guide: [Developer mode and MCP apps in ChatGPT](https://help.openai.com/en/articles/12584461-developer-mode-and-mcp-apps-in-chatgpt).

### Codex

```bash
codex mcp add formbase --url https://api.formbase.so/api/mcp
codex mcp login formbase
```

The second command opens the formbase sign-in page.

### Cursor

Add this to [`.cursor/mcp.json`](.cursor/mcp.json) in your project, or to `~/.cursor/mcp.json` for every project:

```json
{
  "mcpServers": {
    "formbase": {
      "url": "https://api.formbase.so/api/mcp"
    }
  }
}
```

### Cline

Cline connects with an API token (see [below](#scripts-and-headless-agents-use-an-api-token) for how to create one). Add this to Cline's `cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "formbase": {
      "type": "streamableHttp",
      "url": "https://api.formbase.so/api/mcp",
      "headers": {
        "Authorization": "Bearer fb_YOUR_TOKEN"
      }
    }
  }
}
```

Or ask Cline to install it: [`llms-install.md`](llms-install.md) has the steps it follows.

### Grok Build

This repository is also a Grok Build plugin: [`.grok-plugin/plugin.json`](.grok-plugin/plugin.json), the [`.mcp.json`](.mcp.json) server config and one skill, [`formbase-requests`](skills/formbase-requests/SKILL.md). The plugin runs nothing on your machine. It calls one network endpoint, `https://api.formbase.so/api/mcp`, and signs in with formbase OAuth, or with an API token you add as a header.

### Other tools

Any tool that supports remote MCP servers with sign-in works the same way: look for where it adds a server by URL. The [connect guide](https://docs.formbase.so/guides/ai-agents/connect/) links each tool's own instructions.

### Sign in and pick a workspace

The first time the agent uses formbase, your AI tool opens a formbase page in the browser. Sign in, pick the workspace, and click **Authorize**.

A connection reaches that one workspace only. To use another workspace, add formbase a second time and pick the other one.

To check that it works, ask:

```text
List my formbase forms and whether each one is published.
```

To remove the connection, open **OAuth and API Keys** in the formbase workspace sidebar. Under **Connected apps**, click the trash icon next to the connection and confirm with **Disconnect**.

### Scripts and headless agents: use an API token

Anything that cannot open a browser, such as a script, a CI job or a headless agent, sends an API token as a header instead.

1. Open **OAuth and API Keys** in your workspace sidebar and click **Create token** ([API tokens](https://docs.formbase.so/developers/api-tokens/)).
2. Copy the token. It starts with `fb_`.
3. Send it as an `Authorization: Bearer` header.

A token reaches exactly one workspace, like an OAuth connection. It expires 30 days after you create it and cannot be extended, so plan to replace it. Keep it out of git.

Claude Code, from the command line:

```bash
claude mcp add --transport http formbase https://api.formbase.so/api/mcp \
  --header "Authorization: Bearer fb_YOUR_TOKEN"
```

Or in a config file. This is `.mcp.json` for Claude Code; Cursor takes the same `headers` object in its `mcp.json`:

```json
{
  "mcpServers": {
    "formbase": {
      "type": "http",
      "url": "https://api.formbase.so/api/mcp",
      "headers": {
        "Authorization": "Bearer fb_YOUR_TOKEN"
      }
    }
  }
}
```

Other tools take the same URL and header. See their documentation for where.

## Try it

Send a request to one person. Use the name of one of your published forms:

```text
Send a formbase request with the "Supplier onboarding" form to Ada Lovelace (ada@acme.example). Fill in the company name "Analytical Engines Ltd" and lock it so it cannot be changed. Use supplier-2041 as the external ID. Don't send the invitation email; give me the link and I will send it myself.
```

The agent reads the form's field keys with `fields_list`, creates the request with `request_create`, and gives you the request link.

If the form has **Send reminders** on, formbase still emails the recipient its scheduled reminders. To prevent that, also say _no reminders_.

The agent is not told when the recipient submits, so ask it later:

```text
Has the formbase request supplier-2041 been answered? Show me the answers.
```

The agent finds the request by its external ID with `request_list`, then reads it with `request_get`. Once the request is completed, the answers come back keyed by field key.

Step by step: [Send a request with an AI agent](https://docs.formbase.so/guides/ai-agents/send-a-request/) and [Build a form with an AI agent](https://docs.formbase.so/guides/ai-agents/build-a-form/).

## Tools

Every tool is on `tools/list` with its full input schema as soon as an AI tool connects. The [MCP server reference](https://docs.formbase.so/developers/mcp-server/) describes them in detail.

### Requests

Ask one named person for information and read the result.

- `fields_list`: list the field keys a request can address on a published form. Call it before `request_create`.
- `request_create`: assign a published form to one recipient, with prefill, locked fields, context, delivery, expiry and an optional callback.
- `request_get`: read one request by its ID. Returns the status, the timeline and, once completed, the answers keyed by field key.
- `request_list`: list requests for a form or a workspace, filtered by status, outcome or your own external ID.
- `request_cancel`: withdraw a pending request.
- `request_remind`: send the recipient a reminder email now.
- `request_replayCallback`: send a callback again when it never reached your endpoint.
- `document_create`: reserve an upload for a file a request hands to its recipient. One upload can serve any number of requests.

### Forms

- `form_list`, `form_get`, `form_create`, `form_update`: find, read, create and rename forms, and set the folder, emoji, cover and logo.
- `form_publish`, `form_unpublish`: start and stop accepting submissions. `form_publish` is safe to call twice.
- `form_delete`, `form_restore`: move a form to the trash, which also revokes its share links, and bring it back.

`form_get` returns the questions of the last published version. For a draft, read the content with `editor_getDocument`.

### Editor

- `editor_getDocument`: read the full structure of a form.
- `editor_updateElement`, `editor_deleteElement`: edit, move or remove a block.
- `editor_formatText`: format text inside a block.
- `editor_setLogic`, `editor_testLogic`: change the rule of a logic block and test it. Create the block with `editor_insertLogic`.

Each block type has its own insert tool with a precise schema:

- **Questions:** `editor_insertTextQuestion`, `editor_insertContactQuestion`, `editor_insertNumberQuestion`, `editor_insertDateQuestion`, `editor_insertTimeQuestion`, `editor_insertRadioQuestion`, `editor_insertCheckboxQuestion`, `editor_insertSelectQuestion`, `editor_insertPictureChoiceQuestion`, `editor_insertSwitchQuestion`, `editor_insertRatingQuestion`, `editor_insertLinearScaleQuestion`, `editor_insertRankingQuestion`, `editor_insertMatrixQuestion`, `editor_insertFileQuestion`, `editor_insertSignatureQuestion`, `editor_insertPaymentQuestion`, `editor_insertScheduleAppointmentQuestion`
- **Decision:** `editor_insertDecisionQuestion` inserts the approve / decline / changes choice whose answer becomes the request outcome. A radio question built by hand never produces one.
- **Content:** `editor_insertHeader`, `editor_insertParagraph`, `editor_insertImage`, `editor_insertList`, `editor_insertTable`, `editor_insertRow`, `editor_insertEmbedded`, `editor_insertPageDivider`, `editor_insertDocumentsBlock`
- **Data and logic:** `editor_insertHiddenField`, `editor_insertCalculatedField`, `editor_insertVariable`, `editor_insertRepeatingGroup`, `editor_insertLogic`

### Sharing and results

- `formShareLink_list`, `formShareLink_create`, `formShareLink_update`: manage share links, also on a custom domain.
- `formSubmission_list`: list a form's submissions, partial and completed.
- `formAnalytics_get`: views, submissions, completion rate, and breakdowns by device, country, browser and source.

### Appearance, settings and translations

- `formTheme_get`, `formTheme_set`: light and dark themes.
- `formSettings_get`, `formSettings_update`: notification emails, completion redirect, password, retention and language.
- `translationLanguage_list`, `translationDraft_get`, `translationDraft_update`, `translationDraft_publish`, `translationLanguage_delete`: translate a form as a draft, then publish it.

### Workspace

- `workspace_list`: the one workspace your connection reaches, with its ID.
- `workspaceFolder_list`, `workspaceFolder_create`, `workspaceFolder_update`, `workspaceFolder_delete`: manage folders.

### Built-in guides

Two tools return documentation instead of doing work:

- `load_skill` loads a guide on one topic: `requests`, `question-types`, `logic-rules`, `editing-flows`, `form-best-practices`, `form-themes`, `form-settings`, `analytics` or `toon-format`.
- `load_tools` loads usage notes for a group of tools, called a catalog: `request-lifecycle`, `editor-inserts`, `editor-actions`, `form-lifecycle`, `form-appearance`, `form-behavior`, `form-sharing`, `form-translations`, `form-data` or `workspace-management`.

Every guide and catalog is also an MCP resource at `skill://<name>`, such as `skill://requests`. The server also serves four prompts: `identity`, `capabilities`, `data_tools` and `editor_tools`.

### Tools that ask first

Six tools are marked destructive, because undoing them takes another call or is not possible: `form_delete`, `form_unpublish`, `workspaceFolder_delete`, `editor_deleteElement`, `translationLanguage_delete` and `request_cancel`. Most AI tools ask you before running them. The prompt comes from your AI tool, so check its approval settings if you need a hard stop.

Two actions cannot be undone:

- `workspaceFolder_delete` permanently deletes the folder, its subfolders and every form inside.
- `formShareLink_update` with `revoked: true` permanently disables a share link. This tool is not marked destructive, so your AI tool may not ask first.

## Limits

- **Rate limit.** 120 tool calls per minute per token, shared with the [REST API](https://docs.formbase.so/developers/rest-api/). Only `tools/call` counts. Over the limit, the call returns a failed tool result with `RATE_LIMITED` and a `retryAfterMs`. `request_create` and `document_create` also share a second limit of 60 calls per minute per token.
- **Monthly allowance.** Each `request_create` spends one unit of the monthly allowance of the workspace owner's plan, whether or not the recipient ever answers. A test request (`test: true`) spends nothing. When the allowance is spent, `request_create` fails with `MONTHLY_ALLOWANCE_REACHED`.
- **Paid plans.** Emailing a recipient (the invitation and reminders) and `formAnalytics_get` need the Pro or Business plan. See [plans and pricing](https://docs.formbase.so/subscription-billing/plans-pricing/).
- **No notification when a recipient submits.** The server never calls your agent. Ask again later, or pass a `callbackUrl` to `request_create` so formbase calls your endpoint ([callbacks](https://docs.formbase.so/requests/callbacks/)).
- **No file uploads through a tool call.** Images are set by URL: an `http(s)://` URL or a `data:image` URI. A document for a request is the exception: `document_create` returns an upload URL you `PUT` the file to within one hour. PDF and images only, 25 MB per file.
- **No workspace AI skills.** Skills you write in formbase only work in formbase's built-in AI chat. The server's own guides (`load_skill`) work over MCP.

## Protocol details

For anyone who writes their own MCP client or debugs a connection.

- **Transport.** Streamable HTTP, `POST` only. Every response is JSON. There is no event stream and no session ID, so each call stands alone. As the MCP spec requires, send `Content-Type: application/json` and `Accept: application/json, text/event-stream`.
- **Protocol version.** `2025-11-25`.
- **Sign-in.** Every call needs `Authorization: Bearer <token>`, `initialize` included. Without one, the server answers `401` with a `WWW-Authenticate` header that points to `https://api.formbase.so/.well-known/oauth-protected-resource`. From there a client finds the OAuth 2.1 server, which requires PKCE (S256) and supports dynamic client registration. The [reference](https://docs.formbase.so/developers/mcp-server/#oauth) lists each step.
- **Tokens.** Both kinds reach one workspace and the same tools.
  - An API token starts with `fb_` and lasts 30 days from creation.
  - An OAuth access token starts with `fbo_` and lasts 1 hour. Your AI tool renews it with a refresh token, which lasts 30 days and is replaced on every use.
- **Results.** A tool result is one text item that holds JSON. A failure has `success: false` and an `error` message. Request tools add a reason code such as `UNKNOWN_FIELD_KEY` under `details`, and a `suggestion` that says how to fix the call. [Troubleshooting requests](https://docs.formbase.so/requests/troubleshooting/) explains the common ones.
- **Browsers.** The server allows cross-origin calls from formbase's own sites only, so a web page on another origin cannot call it directly.

## Docs

- [MCP server reference](https://docs.formbase.so/developers/mcp-server/): every tool, OAuth for your own client, and limits
- [Connect an AI agent](https://docs.formbase.so/guides/ai-agents/connect/)
- [Build a form with an AI agent](https://docs.formbase.so/guides/ai-agents/build-a-form/)
- [Send a request with an AI agent](https://docs.formbase.so/guides/ai-agents/send-a-request/)
- [Requests overview](https://docs.formbase.so/requests/overview/), [field keys](https://docs.formbase.so/requests/field-keys/), and [callbacks](https://docs.formbase.so/requests/callbacks/)
- [API tokens](https://docs.formbase.so/developers/api-tokens/) and the [REST API](https://docs.formbase.so/developers/rest-api/)
- [Privacy policy](https://docs.formbase.so/legal/privacy-policy/) and [terms of service](https://docs.formbase.so/legal/terms-of-service/)

## Report a problem

- Email [support@formbase.so](mailto:support@formbase.so).
- Or [open an issue](https://github.com/formbaseso/formbase-mcp/issues/new/choose) in this repository. Say which AI tool you use, how it signs in, the tool it called, and the error it got back.

Issues are public. Never paste a token (`fb_...` or `fbo_...`) or a recipient's answers into one; email us instead. To report a security problem, see [SECURITY.md](SECURITY.md).

Fixes to these docs are welcome as pull requests.

## License

The files in this repository are MIT licensed; see [LICENSE](LICENSE). Using the formbase service is covered by the [terms of service](https://docs.formbase.so/legal/terms-of-service/).
