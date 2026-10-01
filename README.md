# formbase MCP server

<img src="logo.png" alt="formbase logo" width="96" align="right">

The hosted MCP server for [formbase.so](https://formbase.so). formbase collects and verifies information from customers for workflows and AI agents. Your agent creates a request for one person. That person, the recipient, gets a branded form with what you already know filled in, completes it on any device without an account, and formbase hands the answers back under field keys your agent can read. Over the same connection an agent can also build, publish, translate and share forms, and read their submissions.

This repository holds documentation and example configs only. There is nothing to install or run. The server lives at:

```text
https://api.formbase.so/api/mcp
```

It speaks MCP (the Model Context Protocol, the standard AI tools use to call other apps) over streamable HTTP. Most AI tools sign in with OAuth: you paste the URL, sign in to formbase in the browser, and pick a workspace. Scripts and headless agents send an API token instead.

## Connect your AI tool

Leave any client ID, secret or token field empty. The tool finds the formbase sign-in page from the server and registers itself.

### Claude Code

```bash
claude mcp add --transport http formbase https://api.formbase.so/api/mcp
```

Then type `/mcp` in Claude Code to sign in.

To share the server with everyone who works on a project, commit a `.mcp.json` to the project instead (this repository has a copy: [`.mcp.json`](.mcp.json)):

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

### Claude desktop (and claude.ai)

In Claude, open **Customize** › **Connectors** › **+** › **Add custom connector** and paste the URL. A connector added on claude.ai also works in the desktop and mobile apps. On Team and Enterprise, an owner adds it first under **Organization settings** › **Connectors**; on Free you can add one custom connector.

### Cursor

Add this to `.cursor/mcp.json` in your project, or to `~/.cursor/mcp.json` for every project (this repository has a copy: [`.cursor/mcp.json`](.cursor/mcp.json)):

```json
{
  "mcpServers": {
    "formbase": {
      "url": "https://api.formbase.so/api/mcp"
    }
  }
}
```

### VS Code

Run **MCP: Add Server** from the Command Palette, choose **HTTP**, and paste the URL. See [VS Code's MCP guide](https://code.visualstudio.com/docs/agent-customization/mcp-servers) for where it saves the server.

### Other tools

Any tool that supports remote MCP servers with sign-in works the same way: look for where it adds a server by URL. The [connect guide](https://docs.formbase.so/guides/ai-agents/connect/) also covers ChatGPT and Codex.

### Sign in and pick a workspace

The first time the agent uses formbase, your AI tool opens a formbase page in the browser. Sign in, pick the workspace, and click **Authorize**. The connection reaches that one workspace only. To use another workspace, add formbase a second time and pick the other one.

To check that it works, ask:

```text
List my formbase forms and whether each one is published.
```

To remove the connection, open **OAuth and API Keys** in the formbase workspace sidebar and click **Disconnect** next to it under **Connected apps**.

### Scripts and headless agents: use an API token

A client that cannot open a browser, such as a script, a CI job or a headless agent, sends an API token as a header instead. Create one under **OAuth and API Keys** in your workspace sidebar ([API tokens](https://docs.formbase.so/developers/api-tokens/)). It starts with `fb_`. Keep it out of git.

Claude Code, from the command line:

```bash
claude mcp add --transport http formbase https://api.formbase.so/api/mcp \
  --header "Authorization: Bearer fb_YOUR_TOKEN"
```

Or in a project's `.mcp.json`:

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

Cursor, in `.cursor/mcp.json` or `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "formbase": {
      "url": "https://api.formbase.so/api/mcp",
      "headers": {
        "Authorization": "Bearer fb_YOUR_TOKEN"
      }
    }
  }
}
```

Other clients take the same URL and header; see their documentation for where. A token reaches exactly one workspace, like an OAuth connection.

## Try it

Send a request to one person. Use the name of one of your published forms:

```text
Send a formbase request with the "Supplier onboarding" form to Ada Lovelace (ada@acme.example). Fill in the company name "Analytical Engines Ltd" and lock it so she cannot change it. Use supplier-2041 as the external ID. Don't email her; give me the link and I will send it myself.
```

The agent reads the form's field keys with `fields_list`, creates the request with `request_create`, and gives you the link. The agent is not told when the recipient submits, so ask it later:

```text
Has the formbase request supplier-2041 been answered? Show me the answers.
```

It calls `request_get` and shows the answers, keyed by field key. Step by step: [Send a request with an AI agent](https://docs.formbase.so/guides/ai-agents/send-a-request/) and [Build a form with an AI agent](https://docs.formbase.so/guides/ai-agents/build-a-form/).

## Tools

Every tool is listed on `tools/list` when a client connects. Clients that load schemas on demand, like Claude Code, fetch a tool's full schema when a task needs it. `load_tools` returns fuller documentation for a catalog of tools, and `load_skill` loads a domain guide.

### Core tools

The tools most tasks start with.

| Tool                                                                                              | What it does                                                                                                                                             |
| ------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `form_list`                                                                                       | List forms in a workspace. Supports folder filter, fuzzy name search, and cursor pagination.                                                             |
| `form_get`                                                                                        | Get full details for a form: questions, cover, logo, and a live preview URL.                                                                             |
| `form_create`                                                                                     | Create a new empty form in a workspace. Returns a preview URL for live editing.                                                                          |
| `form_update`                                                                                     | Update form metadata: name, folder, emoji, cover, or logo.                                                                                               |
| `form_delete`                                                                                     | Move a form to the trash and revoke its share links.                                                                                                     |
| `form_publish`                                                                                    | Publish a form so it can accept submissions. Safe to call twice.                                                                                         |
| `workspace_list`                                                                                  | List all workspaces your token can reach.                                                                                                                |
| `workspaceFolder_list`                                                                            | List folders in a workspace.                                                                                                                             |
| `formSubmission_list`                                                                             | List a form's submissions, partial and completed, with pagination.                                                                                       |
| `fields_list`                                                                                     | List the field keys a request can address on a published form, each with its value type, option keys, and a usage line. Call it before `request_create`. |
| `request_create`                                                                                  | Assign a published form to one named recipient: prefill, locked fields, context, delivery, expiry, and an optional callback.                             |
| `request_get`                                                                                     | Read one request: status, timeline, and, once completed, answers keyed by field key, plus display text.                                                  |
| `request_list`                                                                                    | List requests for a form or a whole workspace, filtered by status, outcome, or your own external id.                                                     |
| `editor_getDocument`                                                                              | Get the full structure of a form: all blocks, their types, and properties.                                                                               |
| `editor_updateElement`                                                                            | Edit a block: replace its text, change properties (title, required and so on), or move it.                                                               |
| `editor_deleteElement`                                                                            | Remove a block from the form.                                                                                                                            |
| `editor_insertTextQuestion`                                                                       | Insert a short or long text question. Each question type has its own insert tool with a precise schema.                                                  |
| `editor_insertContactQuestion`                                                                    | Insert an email, phone number, or website URL question.                                                                                                  |
| `editor_insertNumberQuestion`, `editor_insertDateQuestion`                                        | Insert a number or date question.                                                                                                                        |
| `editor_insertRadioQuestion`, `editor_insertCheckboxQuestion`, `editor_insertSelectQuestion`      | Insert a single-choice (radio), multiple-choice (checkbox), or dropdown question.                                                                        |
| `editor_insertDecisionQuestion`                                                                   | Insert the decision question: the approve / decline / changes choice whose answer becomes a request outcome. A radio built by hand never produces one.   |
| `editor_insertRatingQuestion`, `editor_insertLinearScaleQuestion`                                 | Insert a star rating or a linear scale question.                                                                                                         |
| `editor_insertHeader`, `editor_insertParagraph`, `editor_insertImage`, `editor_insertPageDivider` | Insert content that is not a question: headings, paragraphs, images, and page dividers.                                                                  |
| `load_tools`                                                                                      | Load documentation for a tool catalog: schemas and usage patterns for grouped tools.                                                                     |
| `load_skill`                                                                                      | Load a domain guide (themes, question types, logic rules and more).                                                                                      |

### Tool catalogs

These tools are on `tools/list` too. Run `load_tools` with a catalog name for fuller documentation, then call the tools directly.

| Catalog                | Tools                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `form-data`            | `formAnalytics_get`: views, submissions, completion rate, and breakdowns by device, country, browser and source. Needs the Pro or Business plan; returns `UPGRADE_REQUIRED` otherwise.                                                                                                                                                                                                                                                                                                                                                                                                               |
| `form-appearance`      | `formTheme_get`, `formTheme_set`, `form_update`: light and dark themes, plus the cover and logo.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `form-behavior`        | `formSettings_get`, `formSettings_update`: notification emails, completion redirect, password, retention, language, payment.                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `form-sharing`         | `formShareLink_list`, `formShareLink_create`, `formShareLink_update`: share links, with custom domain support.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `form-translations`    | `translationLanguage_list`, `translationDraft_get`, `translationDraft_update`, `translationDraft_publish`, `translationLanguage_delete`: translate a form as a draft, then publish it.                                                                                                                                                                                                                                                                                                                                                                                                               |
| `form-lifecycle`       | `form_unpublish`, `form_restore`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `workspace-management` | `workspaceFolder_create`, `workspaceFolder_update`, `workspaceFolder_delete`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `editor-actions`       | `editor_formatText`, `editor_setLogic`, `editor_testLogic`: text formatting, writing logic, and testing it.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `request-lifecycle`    | `fields_list`, `request_create`, `request_get`, `request_list`, `request_cancel`, `request_remind`, `request_replayCallback`, `document_create`: the whole request surface. Withdraw a pending request, send the recipient a reminder, replay a callback that never landed, and reserve an upload for a document you hand to one recipient.                                                                                                                                                                                                                                                          |
| `editor-inserts`       | The other insert tools: `editor_insertTimeQuestion`, `editor_insertSwitchQuestion`, `editor_insertFileQuestion`, `editor_insertSignatureQuestion`, `editor_insertDocumentsBlock`, `editor_insertMatrixQuestion`, `editor_insertRankingQuestion`, `editor_insertPaymentQuestion`, `editor_insertScheduleAppointmentQuestion`, `editor_insertPictureChoiceQuestion`, `editor_insertEmbedded`, `editor_insertTable`, `editor_insertList`, `editor_insertRow`, `editor_insertCalculatedField`, `editor_insertHiddenField`, `editor_insertRepeatingGroup`, `editor_insertLogic`, `editor_insertVariable`. |

### Skills, resources and prompts

Skills are built-in guides an agent loads with `load_skill`: `question-types`, `logic-rules`, `editing-flows`, `form-best-practices`, `form-themes`, `form-settings`, `analytics`, `toon-format`, and `requests` (field keys, prefill value shapes, delivery, per-request documents, callbacks, polling, and error recovery).

Every skill and tool catalog is also an MCP resource at `skill://<name>`, such as `skill://requests`. The server also serves three prompts: `identity`, `editor_tools`, and `data_tools`.

### Tools that ask first

These tools are marked destructive, because undoing them takes another call or is not possible: `form_delete`, `form_unpublish`, `workspaceFolder_delete`, `editor_deleteElement`, `translationLanguage_delete`, and `request_cancel`. Most AI tools ask you before running them. The prompt comes from your AI tool, so check its approval settings if you need a hard stop.

## Limits

- **Rate limit.** 120 tool calls per minute per token, shared with the [REST API](https://docs.formbase.so/developers/rest-api/). Only `tools/call` counts. Over the limit, the call returns a failed tool result with `RATE_LIMITED` and a `retryAfterMs`.
- **Monthly allowance.** Each `request_create` spends one unit of your workspace's monthly allowance, whether or not the recipient ever answers.
- **No file uploads through a tool call.** Images are set by URL. A document for one recipient is the exception: `document_create` returns an upload URL you `PUT` the file to.
- **No workspace AI skills.** Skills you write in formbase only work in formbase's built-in AI chat. The server's own skills (`load_skill`) work over MCP.

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

Issues are public. Never paste a token (`fb_...` or `fbo_...`) or your customers' answers into one; email us instead.

## License

The files in this repository are MIT licensed; see [LICENSE](LICENSE). Using the formbase service is covered by the [terms of service](https://docs.formbase.so/legal/terms-of-service/).
