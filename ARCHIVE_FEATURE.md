# Archive Email Feature

## Overview

The `archive_email` tool allows you to archive emails in Gmail, which removes them from your inbox while keeping them accessible via search, "All Mail", and any other labels.

## What is Archiving?

Archiving is Gmail's way of cleaning up your inbox without deleting emails:
- **Removes** the "INBOX" label from the email
- **Keeps** the email in "All Mail"
- **Preserves** all other labels on the email
- **Maintains** full searchability
- **Does NOT delete** the email

Think of it as "filing away" emails you've dealt with but might need later.

## Tool: `archive_email`

### Parameters

- `message_id` (required): The message ID or thread ID to archive
- `is_thread` (optional, default: false): Whether to archive entire thread or just one message

### Returns

```json
{
  "success": true,
  "messageId": "18c3f2a4b5d6e7f8",
  "message": "Archived 3 messages in thread 18c3f2a4b5d6e7f8",
  "isThread": true,
  "note": "Email archived (removed from INBOX but still accessible via search and other labels)"
}
```

## Usage Examples

### Archive a Single Message

```json
{
  "tool": "archive_email",
  "arguments": {
    "message_id": "18c3f2a4b5d6e7f8",
    "is_thread": false
  }
}
```

### Archive Entire Thread

```json
{
  "tool": "archive_email",
  "arguments": {
    "message_id": "18c3f2a4b5d6e7f8",
    "is_thread": true
  }
}
```

## Common Use Cases

### 1. Clean Up Old Emails

```javascript
// Find emails older than 30 days
search_overview("older_than:30d")

// Archive each thread
for (thread in results) {
  archive_email(thread.threadId, true)
}
```

### 2. Archive After Reading

```javascript
// Find unread emails
search_overview("is:unread")

// Read the content
fetch_email_bodies([threadId])

// After reading, mark as read and archive
remove_label(threadId, "UNREAD", true)
archive_email(threadId, true)
```

### 3. Archive Newsletters/Promotions

```javascript
// Find promotional emails
search_overview("category:promotions")

// Archive them all
for (thread in results) {
  archive_email(thread.threadId, true)
}
```

### 4. Archive Completed Tasks

```javascript
// Find emails with "Done" label
search_overview("label:Done")

// Archive to clear inbox
for (thread in results) {
  archive_email(thread.threadId, true)
}
```

## How to Unarchive

To move an email back to your inbox, simply add the INBOX label:

```json
{
  "tool": "add_label",
  "arguments": {
    "message_id": "18c3f2a4b5d6e7f8",
    "label_id": "INBOX",
    "is_thread": true
  }
}
```

## Archive vs Delete

| Action | Archive | Delete |
|--------|---------|--------|
| Removes from inbox | ✅ | ✅ |
| Keeps in All Mail | ✅ | ❌ |
| Can be searched | ✅ | Only in Trash |
| Reversible | ✅ (add INBOX label) | ⚠️ (must recover from Trash) |
| Keeps labels | ✅ | ❌ |
| Permanent | ❌ | ✅ (after 30 days) |

**Recommendation:** Use archive instead of delete for most emails. Only delete spam or truly unwanted emails.

## Performance Notes

- Archiving is very fast (single API call per message)
- When `is_thread: true`, archives all messages in the thread
- Returns count of successfully archived messages
- Failed individual messages don't stop the overall operation

## Workflow Integration

### Inbox Zero Workflow

1. **Process inbox** - Read emails using `search_overview("in:inbox")`
2. **Take action** - Reply (create_draft), label, star, etc.
3. **Archive** - Remove from inbox using `archive_email`
4. **Repeat** - Keep inbox empty

### Weekly Cleanup

```javascript
// Archive all read emails older than 7 days
search_overview("is:read older_than:7d in:inbox")
// Archive each result
archive_email(threadId, true)
```

## Technical Details

- Archives by removing the "INBOX" label using Gmail's modify message API
- Uses `gmail.ModifyMessageRequest` with `RemoveLabelIds: ["INBOX"]`
- Works on both individual messages and entire threads
- Thread mode processes each message in the thread sequentially
- Returns success count for thread operations

## See Also

- [LABEL_MANAGEMENT.md](./LABEL_MANAGEMENT.md) - Complete label management guide
- [LABELS_QUICKSTART.md](./LABELS_QUICKSTART.md) - Quick start guide
- [SEARCH_OVERVIEW.md](./SEARCH_OVERVIEW.md) - Email search guide
