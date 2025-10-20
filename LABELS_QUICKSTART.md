# Label Management Quick Start

## What's New

Gmail MCP Server now supports full label/tag management! You can:
- ⭐ Star important emails
- 🏷️ Add custom labels to organize emails
- 🗑️ Remove labels from emails
- ✨ Create new custom labels
- 📋 List all available labels
- 📦 Archive emails to clear your inbox

## ⚠️ Important: Re-authentication Required

If you previously used this server, you need to re-authenticate with expanded permissions:

```bash
# Delete your old token
rm ~/.auto-gmail/token.json  # Mac/Linux
# or
del %APPDATA%\auto-gmail\token.json  # Windows

# Then restart the server - it will prompt for new OAuth
./gmail-mcp-server
```

## Quick Examples

### Archive an Email
```json
// 1. Search for emails to archive
{"tool": "search_overview", "arguments": {"query": "older_than:30d"}}

// 2. Archive entire thread
{"tool": "archive_email", "arguments": {
  "message_id": "18c3f2a4b5d6e7f8",
  "is_thread": true
}}
```

### Star an Email
```json
// 1. Search for the email
{"tool": "search_overview", "arguments": {"query": "from:important@example.com"}}

// 2. Star it (use the threadId or messageId from results)
{"tool": "add_label", "arguments": {
  "message_id": "18c3f2a4b5d6e7f8",
  "label_id": "STARRED"
}}
```

### Create and Use Custom Labels
```json
// 1. Create a new label
{"tool": "create_label", "arguments": {"label_name": "Important Clients"}}
// Returns: {"labelId": "Label_123456", ...}

// 2. Apply it to an email thread
{"tool": "add_label", "arguments": {
  "message_id": "18c3f2a4b5d6e7f8",
  "label_id": "Label_123456",
  "is_thread": true
}}
```

### Mark Entire Thread as Read
```json
{"tool": "remove_label", "arguments": {
  "message_id": "18c3f2a4b5d6e7f8",
  "label_id": "UNREAD",
  "is_thread": true
}}
```

### View All Your Labels
```json
{"tool": "list_labels", "arguments": {}}
```

## Common System Labels

Just use these IDs directly with `add_label` or `remove_label`:

- `STARRED` - Star/unstar emails
- `IMPORTANT` - Mark as important
- `UNREAD` - Mark as unread/read
- `TRASH` - Move to trash
- `INBOX` - Add to inbox
- `SPAM` - Mark as spam

## Workflow Example: Organize Client Emails

```javascript
// 1. List all labels to see what you have
list_labels()

// 2. Create a new label for clients
create_label("VIP Clients")
// Get the label ID from response: "Label_789"

// 3. Search for client emails
search_overview("from:@importantclient.com")

// 4. Add the label to each thread
add_label(threadId, "Label_789", true)  // true = entire thread

// 5. Also star them for quick access
add_label(threadId, "STARRED", true)
```

## See Full Documentation

For complete details, examples, and all features, see:
- **[LABEL_MANAGEMENT.md](./LABEL_MANAGEMENT.md)** - Complete guide with all features
- **[SEARCH_OVERVIEW.md](./SEARCH_OVERVIEW.md)** - Efficient email searching

## New Tools Summary

| Tool | Purpose |
|------|---------|
| `list_labels` | View all available labels |
| `add_label` | Add a label to email/thread |
| `remove_label` | Remove a label from email/thread |
| `create_label` | Create a new custom label |
| `archive_email` | Archive email (remove from inbox) |

Happy organizing! 🎉
