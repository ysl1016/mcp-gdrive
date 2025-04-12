# Google Drive MCP Server (Fixed Version)

This is a fixed version of the MCP server that integrates with Google Drive to allow listing, reading, and searching files, as well as the ability to read and write to Google Sheets. The key improvement in this version is proper stdout/stderr handling that prevents JSON parsing errors in the Claude app.

This project includes code originally developed by Anthropic, PBC, licensed under the MIT License from [this repo](https://github.com/modelcontextprotocol/servers/tree/main/src/gdrive), with further improvements by [isaacphi/mcp-gdrive](https://github.com/isaacphi/mcp-gdrive).

## What's Fixed

This version fixes the following issues:

1. **JSON Parsing Error**: Redirects all console logs to stderr to prevent the "Unexpected token 'S', "Starting server" is not valid JSON" error in Claude.
2. **Reduced Logging**: Minimizes verbose logging that might interfere with the MCP protocol communication.
3. **Improved Reliability**: Better error handling and separation of log messages from the JSON-RPC communication channel.

## Components

### Tools

- **gdrive_search**

  - **Description**: Search for files in Google Drive.
  - **Input**:
    - `query` (string): Search query.
    - `pageToken` (string, optional): Token for the next page of results.
    - `pageSize` (number, optional): Number of results per page (max 100).
  - **Output**: Returns file names and MIME types of matching files.

- **gdrive_read_file**

  - **Description**: Read contents of a file from Google Drive.
  - **Input**:
    - `fileId` (string): ID of the file to read.
  - **Output**: Returns the contents of the specified file.

- **gsheets_read**

  - **Description**: Read data from a Google Spreadsheet with flexible options for ranges and formatting.
  - **Input**:
    - `spreadsheetId` (string): The ID of the spreadsheet to read.
    - `ranges` (array of strings, optional): Optional array of A1 notation ranges (e.g., `['Sheet1!A1:B10']`). If not provided, reads the entire sheet.
    - `sheetId` (number, optional): Specific sheet ID to read. If not provided with ranges, reads the first sheet.
  - **Output**: Returns the specified data from the spreadsheet.

- **gsheets_update_cell**
  - **Description**: Update a cell value in a Google Spreadsheet.
  - **Input**:
    - `fileId` (string): ID of the spreadsheet.
    - `range` (string): Cell range in A1 notation (e.g., `'Sheet1!A1'`).
    - `value` (string): New cell value.
  - **Output**: Confirms the updated value in the specified cell.

### Resources

The server provides access to Google Drive files:

- **Files** (`gdrive:///<file_id>`)
  - Supports all file types
  - Google Workspace files are automatically exported:
    - Docs → Markdown
    - Sheets → CSV
    - Presentations → Plain text
    - Drawings → PNG
  - Other files are provided in their native format

## Getting started

1. [Create a new Google Cloud project](https://console.cloud.google.com/projectcreate)
2. [Enable the Google Drive API](https://console.cloud.google.com/workspace-api/products)
3. [Configure an OAuth consent screen](https://console.cloud.google.com/apis/credentials/consent) ("internal" is fine for testing)
4. Add OAuth scopes `https://www.googleapis.com/auth/drive.readonly`, `https://www.googleapis.com/auth/spreadsheets`
5. In order to allow interaction with sheets and docs you will also need to enable the [Google Sheets API](https://console.cloud.google.com/apis/api/sheets.googleapis.com/) and [Google Docs API](https://console.cloud.google.com/marketplace/product/google/docs.googleapis.com) in your workspaces Enabled API and Services section.
6. [Create an OAuth Client ID](https://console.cloud.google.com/apis/credentials/oauthclient) for application type "Desktop App"
7. Download the JSON file of your client's OAuth keys
8. Rename the key file to `gcp-oauth.keys.json` and place into the path you specify with `GDRIVE_CREDS_DIR` (i.e. `/Users/username/.config/mcp-gdrive`)
9. Note your OAuth Client ID and Client Secret. They must be provided as environment variables along with your configuration directory.
10. You will also need to setup a .env file within the project with the following fields. You can find the Client ID and Client Secret in the Credentials section of the Google Cloud Console.

```
GDRIVE_CREDS_DIR=/path/to/config/directory
CLIENT_ID=<CLIENT_ID>
CLIENT_SECRET=<CLIENT_SECRET>
```

Make sure to build the server with either `npm run build` or `npm run watch`.

### Authentication

Next you will need to run `node ./dist/index.js` to trigger the authentication step

You will be prompted to authenticate with your browser. You must authenticate with an account in the same organization as your Google Cloud project.

Your OAuth token is saved in the directory specified by the `GDRIVE_CREDS_DIR` environment variable.

![Authentication Prompt](https://i.imgur.com/TbyV6Yq.png)

### Usage with Claude Desktop App

To integrate this fixed server with the Claude Desktop app, add the following to your app's server configuration (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "gdrive": {
      "command": "npx",
      "args": ["-y", "@ysl1016/mcp-gdrive"],
      "env": {
        "CLIENT_ID": "<CLIENT_ID>",
        "CLIENT_SECRET": "<CLIENT_SECRET>",
        "GDRIVE_CREDS_DIR": "/path/to/config/directory"
      }
    }
  }
}
```

## Troubleshooting

If you're experiencing the "Unexpected token 'S', "Starting server" is not valid JSON" error in Claude, it means the original mcp-gdrive package is sending log messages to stdout instead of stderr, which interferes with the JSON-RPC protocol. This fixed version solves that issue by:

1. Redirecting all console.log outputs to stderr
2. Reducing verbose logging
3. Ensuring proper separation of logs from the JSON communication channel

## License

This MCP server is licensed under the MIT License. This means you are free to use, modify, and distribute the software, subject to the terms and conditions of the MIT License. For more details, please see the LICENSE file in the project repository.