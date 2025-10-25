# Message Reading Guide

## Overview

The Gmail MCP Server provides multiple ways to read email messages, each optimized for different use cases.

## The Problem

Previously, `fetch_email_bodies` only returned the **first message** in a thread, making it impossible to read replies and follow-up messages in email conversations.

## The Solution

Two new tools have been added:

1. **`get_message`** - Read a specific individual message
2. **`get_thread_messages`** - Read ALL messages in a thread with full content

## Tools Comparison

| Tool | Purpose | Returns | Best For |
|------|---------|---------|----------|
| `search_overview` | Find emails | Thread ID, subject, date, last message ID | Quick browsing |
| `fetch_email_bodies` | Get thread content | First message only | Legacy support |
| **`get_message`** | Read one message | Full message content | Reading specific replies |
| **`get_thread_messages`** | Read all messages | All messages in thread | Reading conversations |

## New Tool: `get_message`

Retrieves the full content of a **specific message** by message ID.

### Parameters

- `message_id` (required): The specific message ID (not thread ID)

### Returns

```json
{
  "messageId": "18c3f2a4b5d6e7f8",
  "threadId": "18c3f2a4b5d6e7f0",
  "subject": "Re: Project Update",
  "from": "wendy@example.com",
  "to": "john@example.com",
  "cc": "team@example.com",
  "date": "Fri, 20 Oct 2025 13:32:00 -0700",
  "dateTime": "2025-10-20T13:32:00-07:00",
  "snippet": "Thanks for the update. I've reviewed...",
  "fullBody": "Thanks for the update. I've reviewed the documents and have a few questions...",
  "labelIds": ["INBOX", "UNREAD", "IMPORTANT"],
  "inReplyTo": "<previous-message-id@gmail.com>",
  "references": "<original-message-id@gmail.com> <previous-message-id@gmail.com>",
  "attachments": [...]
}
```

### Example Usage

```json
// Get a specific message (e.g., Wendy's reply)
{
  "tool": "get_message",
  "arguments": {
    "message_id": "18c3f2a4b5d6e7f9"
  }
}
```

### When to Use

- Reading a specific reply in a thread
- Getting details of one particular message
- Following up on a single email

## New Tool: `get_thread_messages`

Retrieves **all messages** in a thread with configurable detail level to control token usage.

### Parameters

- `thread_id` (required): The thread ID
- `summary_only` (optional, default: false): If true, returns only snippets (saves ~80% tokens)
- `max_body_length` (optional, default: 2000): Max characters per message body (reduced from 8000)

### Performance Modes

**Summary Mode** (`summary_only: true`):
- Returns only snippets (~150 chars per message)
- Minimal metadata: messageId, subject, from, to, dateTime, snippet
- Notes if attachments exist (count only, no details)
- **Token usage: ~1-2k tokens** for typical threads
- Perfect for: Quick overview, deciding which messages to read fully

**Full Mode** (`summary_only: false`, default):
- Returns full message bodies (up to `max_body_length`, default 2000 chars)
- Complete metadata: all headers, labels, attachment details
- **Token usage: ~3-6k tokens** with default settings (was ~11k before)
- Perfect for: Reading complete conversations

### Returns (Summary Mode)

```json
{
  "threadId": "18c3f2a4b5d6e7f0",
  "messageCount": 3,
  "messages": [
    {
      "messageId": "18c3f2a4b5d6e7f1",
      "subject": "Project Update",
      "from": "john@example.com",
      "to": "wendy@example.com",
      "dateTime": "2025-10-19T10:00:00-07:00",
      "snippet": "Here's the latest update on the project..."
    },
    {
      "messageId": "18c3f2a4b5d6e7f2",
      "subject": "Re: Project Update",
      "from": "wendy@example.com",
      "to": "john@example.com",
      "dateTime": "2025-10-20T13:32:00-07:00",
      "snippet": "Thanks for the update. I've reviewed...",
      "hasAttachments": true,
      "attachmentCount": 2
    }
  ]
}
```

### Returns (Full Mode)

```json
{
  "threadId": "18c3f2a4b5d6e7f0",
  "messageCount": 3,
  "messages": [
    {
      "messageId": "18c3f2a4b5d6e7f1",
      "subject": "Project Update",
      "from": "john@example.com",
      "to": "wendy@example.com",
      "date": "Thu, 19 Oct 2025 10:00:00 -0700",
      "dateTime": "2025-10-19T10:00:00-07:00",
      "snippet": "Here's the latest update...",
      "fullBody": "Here's the latest update on the project...",
      "labelIds": ["INBOX", "SENT"]
    },
    {
      "messageId": "18c3f2a4b5d6e7f2",
      "subject": "Re: Project Update",
      "from": "wendy@example.com",
      "to": "john@example.com",
      "date": "Fri, 20 Oct 2025 13:32:00 -0700",
      "dateTime": "2025-10-20T13:32:00-07:00",
      "snippet": "Thanks for the update...",
      "fullBody": "Thanks for the update. I've reviewed the documents...",
      "labelIds": ["INBOX", "IMPORTANT"],
      "inReplyTo": "<18c3f2a4b5d6e7f1@gmail.com>"
    },
    {
      "messageId": "18c3f2a4b5d6e7f3",
      "subject": "Re: Project Update",
      "from": "john@example.com",
      "to": "wendy@example.com",
      "date": "Fri, 20 Oct 2025 15:00:00 -0700",
      "dateTime": "2025-10-20T15:00:00-07:00",
      "snippet": "Great questions. Let me address...",
      "fullBody": "Great questions. Let me address each one...",
      "labelIds": ["SENT"]
    }
  ]
}
```

### Example Usage

```json
// Quick summary (lightweight, ~1-2k tokens)
{
  "tool": "get_thread_messages",
  "arguments": {
    "thread_id": "18c3f2a4b5d6e7f0",
    "summary_only": true
  }
}

// Full content with default body length (2000 chars per message, ~3-6k tokens)
{
  "tool": "get_thread_messages",
  "arguments": {
    "thread_id": "18c3f2a4b5d6e7f0"
  }
}

// Full content with custom body length (500 chars per message, ~2-3k tokens)
{
  "tool": "get_thread_messages",
  "arguments": {
    "thread_id": "18c3f2a4b5d6e7f0",
    "summary_only": false,
    "max_body_length": 500
  }
}
```

### When to Use

- Reading an entire email conversation
- Understanding the full context of a thread
- Analyzing back-and-forth discussions
- Getting all replies in chronological order

## Token Usage Optimization

### Choosing the Right Mode

| Scenario | Recommended Mode | Token Usage |
|----------|------------------|-------------|
| "What emails are in this thread?" | `summary_only: true` | ~1-2k tokens |
| "Show me who replied and when" | `summary_only: true` | ~1-2k tokens |
| "I need to read the full conversation" | default (2000 chars) | ~3-6k tokens |
| "Just need first 500 chars of each" | `max_body_length: 500` | ~2-3k tokens |
| "Need complete untruncated content" | `max_body_length: 8000` | ~8-15k tokens |

### Best Practices

1. **Start with summary mode** - See what's in the thread before loading full content
2. **Use get_message for specific replies** - If you know which message you need, fetch just that one
3. **Adjust max_body_length** - Most emails don't need 8000 chars, 2000 is usually enough
4. **Progressive loading** - Summary first, then full content only if needed

### Example: Efficient Thread Reading

```javascript
// Step 1: Get thread summary (cheap, ~1-2k tokens)
get_thread_messages(threadId, summary_only=true)
// Review snippets, see there are 5 messages

// Step 2: Identify which message you need
// "Wendy's reply is message #3"

// Step 3: Get just that message (cheap, ~1-2k tokens)
get_message(message3Id)

// Total: ~2-4k tokens instead of ~11k for full thread!
```

## Common Workflows

### Workflow 1: Read Recent Reply (Optimized)

```javascript
// 1. Search for the thread
search_overview("from:wendy@example.com subject:project")

// 2. Get thread summary first (lightweight, ~1-2k tokens)
get_thread_messages(threadId, summary_only=true)

// 3. See Wendy's latest reply is the 3rd message
// Get just that specific message (another ~1-2k tokens)
get_message(message3Id)

// Total: ~2-4k tokens vs ~11k for full thread
```

### Workflow 2: Read Specific Message

```javascript
// 1. Search for thread
search_overview("subject:meeting")
// Get the thread ID

// 2. Get thread messages to see all message IDs
get_thread_messages(threadId, {summary_only: true})

// 3. Read specific message by its ID
get_message(specificMessageId)
```

### Workflow 3: Analyze Conversation

```javascript
// 1. Find the conversation
search_overview("subject:'Q4 Planning'")

// 2. Get all messages
get_thread_messages(threadId)

// 3. Process each message in the array
for (message in messages) {
  console.log(message.from, message.dateTime, message.fullBody)
}
```

### Workflow 4: Reply to Latest Message

```javascript
// 1. Get all messages in thread
get_thread_messages(threadId)

// 2. Read the last message (latest reply)
latestMessage = messages[messages.length - 1]

// 3. Draft a reply
create_draft(
  to: latestMessage.from,
  subject: latestMessage.subject,
  body: "Your reply...",
  thread_id: threadId
)
```

## Message Fields Explained

| Field | Description |
|-------|-------------|
| `messageId` | Unique message ID |
| `threadId` | Parent thread ID |
| `subject` | Email subject |
| `from` | Sender email address |
| `to` | Primary recipient(s) |
| `cc` | CC'd recipients (optional) |
| `date` | Original date header |
| `dateTime` | Parsed timestamp (RFC3339) |
| `snippet` | Gmail's short preview (150 chars) |
| `fullBody` | Complete email body (markdown formatted, max 8000 chars) |
| `labelIds` | Array of label IDs on this message |
| `inReplyTo` | Message ID this is replying to (optional) |
| `references` | Chain of message IDs in conversation (optional) |
| `replyTo` | Reply-To address if different from From (optional) |
| `attachments` | Array of attachment info (optional) |

## Message Order

- Messages in `get_thread_messages` are returned in **chronological order**
- First message = original email
- Last message = most recent reply
- Use array indexing to access specific positions

## Performance Notes

- Both tools fetch full message content (not just metadata)
- Body content is limited to 8000 characters per message to prevent context overflow
- Longer messages are truncated with a note
- Attachments are listed but not loaded (use `extract_attachment_by_filename` to read them)
- `get_thread_messages` makes one API call regardless of message count

## Migration from `fetch_email_bodies`

**Old way** (only gets first message):
```javascript
fetch_email_bodies([threadId])
// Returns: only the original email
```

**New way** (gets all messages):
```javascript
get_thread_messages(threadId)
// Returns: entire conversation in order
```

## Tips

1. **Use `search_overview` first** - Get thread IDs quickly
2. **Use `get_thread_messages` for conversations** - See the full back-and-forth
3. **Use `get_message` for specific messages** - When you know the exact message ID
4. **Check `dateTime` field** - To understand message order and timing
5. **Look at `inReplyTo`** - To understand message relationships
6. **Array length = message count** - `messages.length` tells you how many replies

## See Also

- [SEARCH_OVERVIEW.md](./SEARCH_OVERVIEW.md) - Efficient email searching
- [LABEL_MANAGEMENT.md](./LABEL_MANAGEMENT.md) - Managing email labels
- [ARCHIVE_FEATURE.md](./ARCHIVE_FEATURE.md) - Archiving emails
