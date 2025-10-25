# Gmail MCP Server

An MCP server that lets AI agents search Gmail threads, understand your email writing style, and create draft emails.

## 1. Get Google Authentication

### Step 1: Create a Google Cloud Project
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Click "Select a project" dropdown at the top
3. Click "New Project"
4. Enter a project name (e.g., "Gmail MCP Server")
5. Click "Create"

### Step 2: Enable Gmail API
1. In your new project, go to "APIs & Services" → "Library"
2. Search for "Gmail API"
3. Click on "Gmail API" and click "Enable"

### Step 3: Create OAuth2 Credentials
1. Go to "APIs & Services" → "Credentials"
2. Click "Create Credentials" → "OAuth Client ID"
3. If prompted, configure the OAuth consent screen:
   - Choose "External" user type
   - Fill in required fields (App name, User support email, Developer email)
   - Add your email to "Test users" section
   - Save and continue through all steps
4. Back in Credentials, click "Create Credentials" → "OAuth Client ID"
5. Choose "Desktop application" as the application type
6. Enter a name (e.g., "Gmail MCP Client")
7. Click "Create"
8. **Important**: Copy the **Client ID** and **Client Secret** from the confirmation dialog (you'll need these for configuration)

### Step 4: Grant OAuth Scopes
When you first run the server, it will open your browser for authorization. The server requests **only these minimal permissions**:

#### What We Request:
- ✅ **Gmail Readonly Access** (`gmail.readonly`)
  - Search and read your email messages
  - Download email attachments  
  - View email metadata (subjects, senders, dates)

- ✅ **Gmail Compose Access** (`gmail.compose`)
  - Create email drafts
  - Update existing drafts
  - Delete drafts
  - **Send emails** (permission granted but not used by this server)

#### What This Server Actualy Implements:
- ✅ **Search and read emails** - Full search capabilities
- ✅ **Extract attachment text** - Safe PDF/DOCX/TXT text extraction
- ✅ **Download attachments** - Download any attachment to local file (no context usage)
- ✅ **Create/update drafts** - Smart draft management with thread awareness
- ❌ **Send emails** - Server doesn't implement sending (though permission is granted)
- ❌ **Delete emails** - Server doesn't implement deletion
- ❌ **Modify labels** - Server doesn't implement label management

## 2. Add to MCP Clients

Build the server first: `go build .`

You'll want to add this to your agent's configuration file:

```json
{
  "mcpServers": {
    "gmail": {
      "command": "C:/path/to/your/auto-gmail.exe",
      "env": {
        "GMAIL_CLIENT_ID": "your_client_id_here.apps.googleusercontent.com",
        "GMAIL_CLIENT_SECRET": "your_client_secret_here",
        "OPENAI_API_KEY": "your_openai_api_key_here"
      }
    }
  }
}
```

### ⚡ **Persistent HTTP Mode (Recommended to Avoid OAuth Popups)**

**Problem**: In stdio mode, Cursor starts a fresh server process each time (and for each tab), causing OAuth popup spam.

**Solution**: Run the server as a persistent HTTP daemon that authenticates once and stays running.

#### Quick Start:
```bash
# Build the server
go build -o gmail-mcp-server

# Start persistent server (OAuth only once!)
./gmail-mcp-server --http

# Or with custom port
./gmail-mcp-server --http 3000
```

#### What Happens:
1. ✅ **OAuth popup appears ONCE** when server starts
2. ✅ **Server runs persistently** on http://localhost:8080  
3. ✅ **No more OAuth popups** - server stays authenticated
4. ✅ **Multiple Cursor tabs/windows** can connect to same server

#### Cursor Configuration (Still Use stdio for now):
```json
{
  "mcpServers": {
    "gmail": {
      "command": "C:/path/to/your/gmail-mcp-server",
      "env": {
        "GMAIL_CLIENT_ID": "your_client_id_here.apps.googleusercontent.com", 
        "GMAIL_CLIENT_SECRET": "your_client_secret_here",
        "OPENAI_API_KEY": "your_openai_api_key_here"
      }
    }
  }
}
```

The difference: **You start the server manually once** instead of letting Cursor start it fresh each time.

#### Server Status:
- Visit http://localhost:8080 to see server status
- Health check: http://localhost:8080/health
- View available tools and configuration examples

### Add to Cursor
- Press `Ctrl+Shift+P` (Windows/Linux) or `Cmd+Shift+P` (Mac)
- Click the MCP-tab
- Click '+ Add new global MCP server'
- Edit config file

### Add to Claude Desktop
- Go to File > Settings > Developer > Edit Config
- Edit Config file

### Manual Configuration Alternative:
You can edit these config files directly if you know where to find them:

- **Cursor:** `C:\Users\[User]\.cursor\mcp.json`
- **Claude Desktop:** `%APPDATA%\Claude\claude_desktop_config.json` (Windows)

## 3. MCP Tools and Resources

**Tools:**
- `search_overview` - Search Gmail with queries like "from:email@example.com" or "subject:meeting" (returns thread IDs and basic metadata)
- `get_thread_messages` - Read all messages in a thread (with configurable detail level)
- `get_message` - Read a specific message by message ID
- `create_draft` - Create email drafts or update existing drafts (AI will request style guide first)
- `extract_attachment_by_filename` - Safely extract text from PDF, DOCX, and TXT attachments using filename
- `download_attachment` - Download any email attachment directly to a local file path without loading it into the context window (perfect for large files, images, videos, or any binary attachments)
- `get_personal_email_style_guide` - Get your email writing style guide (this is a temporary tool, created because most agents do not yet support fetching resources--once agents implement MCP resources better, then thsi tool can be removed)

**Resources:**
- `file://personal-email-style-guide` - Your personal email writing style (auto-generated or manual)

**Prompts:**
- `/generate-email-tone` - Analyze your sent emails to create personalized writing style
- `/server-status` - Show file locations and server status

### Tool Usage Examples

#### Download Attachments
Use `download_attachment` to save email attachments to disk without loading them into the context window:

```
Parameters:
- message_id: "abc123def456" (from search_overview or get_thread_messages results)
- filename: "invoice.pdf" (exact filename from the email)
- local_path: "/Users/john/Downloads/invoice.pdf" (where to save)

Returns metadata only:
{
  "messageId": "abc123def456",
  "filename": "invoice.pdf",
  "mimeType": "application/pdf",
  "savedTo": "/Users/john/Downloads/invoice.pdf",
  "fileSize": 524288,
  "downloadedAt": "2025-10-06T14:30:00Z",
  "success": true
}
```

**When to use:**
- ✅ Large files (videos, images, archives)
- ✅ Binary files you want to save locally
- ✅ Files you'll process with other tools
- ✅ Saving context window tokens

**When to use `extract_attachment_by_filename` instead:**
- ✅ Small text files (PDF, DOCX, TXT) you want to analyze
- ✅ Documents you need to read/search through
- ✅ Content that should be in the conversation

## 4. Personal Email Style Guide

The server will create a style-guide file based on the last 25 emails you've sent, so that newly drafted emails will hopefully sound like you. Honestly, so far LLM-written emails still don't sound very authentic.

**Manual Generation:**
- Run `/generate-email-tone` prompt in your MCP client anytime to regenerate
- The file is saved to your app data directory (see **File Storage Locations** above)

**AI Integration:**
- AI always calls `get_personal_email_style_guide` tool before writing emails
- Ensures consistent personal style across all communications
- Resource also available at `file://personal-email-style-guide`

## 5. Alternative way to Setup Environment Variables

If you want to run this MCP server outside of an agent, you can create a .env file based on the .env.example file and supply the environment variables that way, or export them into your environment prior to running:

```bash
export GMAIL_CLIENT_ID=your_client_id_here.apps.googleusercontent.com
export GMAIL_CLIENT_SECRET=your_client_secret_here
export OPENAI_API_KEY=your_openai_api_key_here
```

## 6. File Storage Locations

The server stores authentication and configuration files in standard application directories:

### File Locations:
- **Windows**: `C:\Users\[YourUsername]\AppData\Roaming\auto-gmail\`
- **Mac**: `~/.auto-gmail/`  
- **Linux**: `~/.auto-gmail/`

### Important Files:
- **`token.json`** - OAuth authentication token (auto-generated)
- **`personal-email-style-guide.md`** - Your email writing style guide (auto-generated or manual)

### Quick Commands:
- Use `/server-status` in your MCP client to see exact file paths
- Delete `token.json` to force re-authentication with updated permissions

## 7. TODOs

- [x] **Improve OAuth login flow** - ✅ **SOLVED!** Use persistent HTTP mode (`./gmail-mcp-server --http`) to avoid OAuth popups. Server authenticates once and stays running.
- [ ] **Full HTTP MCP Transport** - Waiting for mark3labs/mcp-go to expose complete HTTP transport APIs
- [ ] **Better Email Authenticity** - LLM-written emails still don't sound perfectly authentic despite style guides
