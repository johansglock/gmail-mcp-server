# Label Management Features

## Overview

Gmail MCP Server now supports full label/tag management, allowing you to organize emails by adding, removing, and creating custom labels. In Gmail, labels are equivalent to tags or folders in other email systems.

## Important Note: OAuth Scope Update

**The OAuth scope has been updated to `gmail.modify` which includes full read, compose, and label modification permissions.**

If you have previously authenticated with this server, you will need to:
1. Delete your existing token file: `~/.auto-gmail/token.json` (or `%APPDATA%\auto-gmail\token.json` on Windows)
2. Restart the server to trigger a new OAuth flow with the updated permissions

## Available Tools

### 1. `archive_email` - Archive Emails

Archives an email message or thread by removing it from the inbox. The email remains accessible via search, "All Mail", and any other labels it has, but is removed from the inbox view. This is Gmail's standard archive operation.

**Parameters:**
- `message_id` (required): The message ID or thread ID to archive
- `is_thread` (optional, default: false): If true, archives all messages in the thread

**Returns:**
- `success`: Boolean indicating success
- `messageId`: The message/thread ID that was archived
- `message`: Description of what was archived
- `isThread`: Whether entire thread was archived
- `note`: Explanation that email is still accessible

**Example Usage:**
```json
// Archive a single message
{
  "tool": "archive_email",
  "arguments": {
    "message_id": "18c3f2a4b5d6e7f8",
    "is_thread": false
  }
}

// Archive entire thread
{
  "tool": "archive_email",
  "arguments": {
    "message_id": "18c3f2a4b5d6e7f8",
    "is_thread": true
  }
}
```

**Example Response:**
```json
{
  "success": true,
  "messageId": "18c3f2a4b5d6e7f8",
  "message": "Archived 3 messages in thread 18c3f2a4b5d6e7f8",
  "isThread": true,
  "note": "Email archived (removed from INBOX but still accessible via search and other labels)"
}
```

**How It Works:**
- Archiving removes the "INBOX" label from the email
- The email stays in "All Mail" and under any other labels it has
- You can still find it via search
- To unarchive, use `add_label` with `label_id: "INBOX"`

### 2. `list_labels` - List All Labels

Retrieves all available Gmail labels, including both system labels (INBOX, SENT, STARRED, etc.) and user-created custom labels.

**Parameters:** None

**Returns:**
- `id`: Label ID (used for add/remove operations)
- `name`: Display name of the label
- `type`: Either "system" or "user"
- `messagesTotal`: Total messages with this label (if > 0)
- `messagesUnread`: Unread messages with this label (if > 0)

**Example Usage:**
```json
{
  "tool": "list_labels",
  "arguments": {}
}
```

**Example Response:**
```json
[
  {
    "id": "INBOX",
    "name": "INBOX",
    "type": "system",
    "messagesTotal": 523,
    "messagesUnread": 42
  },
  {
    "id": "STARRED",
    "name": "STARRED",
    "type": "system",
    "messagesTotal": 15
  },
  {
    "id": "Label_123456",
    "name": "Work Projects",
    "type": "user",
    "messagesTotal": 87,
    "messagesUnread": 12
  }
]
```

### 3. `add_label` - Add Label to Email

Adds a label/tag to a specific message or an entire thread.

**Parameters:**
- `message_id` (required): The message ID or thread ID
- `label_id` (required): The label ID to add (from `list_labels`)
- `is_thread` (optional, default: false): If true, adds to all messages in the thread

**Common System Labels:**
- `STARRED` - Star an email
- `IMPORTANT` - Mark as important
- `UNREAD` - Mark as unread
- `TRASH` - Move to trash
- `SPAM` - Mark as spam
- Custom label IDs from `list_labels`

**Example Usage:**
```json
{
  "tool": "add_label",
  "arguments": {
    "message_id": "18c3f2a4b5d6e7f8",
    "label_id": "STARRED",
    "is_thread": false
  }
}
```

**Example Response:**
```json
{
  "success": true,
  "messageId": "18c3f2a4b5d6e7f8",
  "labelId": "STARRED",
  "message": "Added label 'STARRED' to message 18c3f2a4b5d6e7f8",
  "isThread": false
}
```

### 4. `remove_label` - Remove Label from Email

Removes a label/tag from a specific message or an entire thread.

**Parameters:**
- `message_id` (required): The message ID or thread ID
- `label_id` (required): The label ID to remove
- `is_thread` (optional, default: false): If true, removes from all messages in the thread

**Example Usage:**
```json
{
  "tool": "remove_label",
  "arguments": {
    "message_id": "18c3f2a4b5d6e7f8",
    "label_id": "UNREAD",
    "is_thread": true
  }
}
```

**Example Response:**
```json
{
  "success": true,
  "messageId": "18c3f2a4b5d6e7f8",
  "labelId": "UNREAD",
  "message": "Removed label 'UNREAD' from 3 messages in thread 18c3f2a4b5d6e7f8",
  "isThread": true
}
```

### 5. `create_label` - Create New Custom Label

Creates a new custom Gmail label that can be used to organize emails.

**Parameters:**
- `label_name` (required): The name for the new label

**Example Usage:**
```json
{
  "tool": "create_label",
  "arguments": {
    "label_name": "Client Proposals"
  }
}
```

**Example Response:**
```json
{
  "success": true,
  "labelId": "Label_987654",
  "name": "Client Proposals",
  "message": "Created label 'Client Proposals' with ID 'Label_987654'"
}
```

## Common Workflows

### Workflow 1: Archive Old Emails

1. Search for old emails: `search_overview` with query "older_than:30d"
2. Review the thread IDs
3. Archive them: `archive_email` with `is_thread: true`

### Workflow 2: Star Important Emails

1. Search for emails: `search_overview` with query "from:boss@company.com"
2. Add star to specific messages: `add_label` with `label_id: "STARRED"`

### Workflow 3: Organize Emails with Custom Labels

1. Create a new label: `create_label` with `label_name: "Q4 Planning"`
2. List labels to get the new label ID: `list_labels`
3. Search for relevant emails: `search_overview` with query "subject:Q4"
4. Add label to threads: `add_label` with the new label ID and `is_thread: true`

### Workflow 4: Mark Thread as Read

1. Find unread threads: `search_overview` with query "is:unread"
2. Remove UNREAD label: `remove_label` with `label_id: "UNREAD"` and `is_thread: true`

### Workflow 5: Batch Label Operations

1. Search for specific emails: `search_overview` with a query
2. Get multiple thread IDs from results
3. Loop through and `add_label` to each thread
4. Use `is_thread: true` to apply to entire conversations

## Tips

- **Always use `list_labels` first** to get the correct label IDs
- **System labels** use ALL_CAPS (INBOX, STARRED, IMPORTANT, UNREAD, TRASH, SPAM)
- **Custom labels** have IDs like "Label_123456"
- **Use `is_thread: true`** to apply labels to entire conversations efficiently
- **Label names are case-sensitive** when creating new labels
- **Labels are hierarchical** - you can create nested labels like "Work/Projects/Q4"

## System Labels Reference

| Label ID | Description | Common Use |
|----------|-------------|------------|
| INBOX | Inbox folder | Main mailbox |
| SENT | Sent emails | Outgoing mail |
| DRAFT | Draft emails | Unfinished emails |
| STARRED | Starred/flagged | Important items |
| IMPORTANT | Gmail's important marker | Priority emails |
| UNREAD | Unread messages | New emails |
| TRASH | Trash/deleted | Deleted emails |
| SPAM | Spam folder | Junk mail |
| CATEGORY_PERSONAL | Personal category | Personal emails |
| CATEGORY_SOCIAL | Social category | Social networks |
| CATEGORY_PROMOTIONS | Promotions category | Marketing emails |
| CATEGORY_UPDATES | Updates category | Notifications |
| CATEGORY_FORUMS | Forums category | Mailing lists |

## Error Handling

- If a label ID doesn't exist, you'll get an error message
- If a message ID doesn't exist, you'll get an error message
- Creating a label with a duplicate name may fail or return the existing label
- Some system labels cannot be created or deleted (INBOX, SENT, etc.)

## Performance Considerations

- `add_label` and `remove_label` with `is_thread: true` will modify all messages in the thread
- For large threads, this may take a few seconds
- The operation returns the count of successfully modified messages
- Failed individual message modifications are logged but don't stop the overall operation
