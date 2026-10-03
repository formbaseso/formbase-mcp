# Installing the formbase MCP server

formbase is a hosted MCP server at `https://api.formbase.so/api/mcp`. There is nothing to clone, build or run locally.

1. Ask the user for a formbase API token. They create one in their formbase workspace: open **OAuth and API Keys** in the sidebar and click **Create token**. The token starts with `fb_`, reaches one workspace and expires 30 days after it is created. Docs: https://docs.formbase.so/developers/api-tokens/
2. Add this entry to `cline_mcp_settings.json`, with the user's token in place of `fb_YOUR_TOKEN`. Keep any servers that are already there.

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

3. Check the connection by calling `workspace_list`. It returns the workspace the token belongs to.

A 401 response means the token is wrong or has expired. Ask the user to create a new one.

What to try next and the full tool list: [README.md](README.md).
