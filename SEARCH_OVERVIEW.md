# Search Overview Feature

## New MCP Tool: `search_overview`

A lightweight email search tool that returns only essential metadata without loading full email content into context.

## Purpose

The `search_overview` tool is perfect for quickly browsing many emails when you only need to see what exists, without consuming context tokens on full email bodies, snippets, attachments, or draft information.

## What It Returns

For each matching thread, returns only:
- **threadId**: The Gmail thread ID
- **lastMessageId**: The ID of the last message in the thread
- **lastMessageDateTime**: The date and time of the last message (RFC3339 format)
- **subject**: The email subject line

## Parameters

- `query` (required): Gmail search query using standard Gmail search operators
- `max_results` (optional): Maximum number of threads to return (default: 10)

## What It Returns

`search_overview` returns lightweight metadata for each thread:

| Field | Description |
|-------|-------------|
| Thread ID | Gmail thread identifier |
| Subject | Email subject line |
| Last Message ID | ID of the most recent message |
| Last Message Date/Time | When the last message was sent (RFC3339) |

## When to Use

**Use `search_overview` when:**
- Finding emails by search criteria
- Browsing many emails efficiently
- Getting thread IDs for further investigation
- Minimizing context token usage (~100-500 tokens)
- Identifying which threads to read with `get_thread_messages`

**For more details, use:**
- `get_thread_messages` - Read all messages in a thread
- `get_message` - Read a specific message by ID

## Example Usage

```javascript
// Quick overview of unread emails
{
  "tool": "search_overview",
  "arguments": {
    "query": "is:unread",
    "max_results": 20
  }
}

// Find recent emails from a specific sender
{
  "tool": "search_overview",
  "arguments": {
    "query": "from:john@example.com newer_than:7d"
  }
}
```

## Example Response

```json
[
  {
    "threadId": "18c3f2a4b5d6e7f8",
    "lastMessageId": "18c3f2a4b5d6e7f9",
    "lastMessageDateTime": "2025-10-18T10:30:00-07:00",
    "subject": "Re: Project Update"
  },
  {
    "threadId": "18c3f2a4b5d6e800",
    "lastMessageId": "18c3f2a4b5d6e801",
    "lastMessageDateTime": "2025-10-18T09:15:00-07:00",
    "subject": "Meeting Notes - Q4 Planning"
  }
]
```

## Implementation Details

- Uses Gmail API's `Format("metadata")` option for efficient data retrieval
- Only fetches message headers (Subject, Date) instead of full message payloads
- Returns the **last** message in each thread (most recent)
- Date/time is derived from Gmail's `InternalDate` field for accuracy
- Falls back to the `Date` header if `InternalDate` is unavailable

## Workflow Integration

A typical workflow might be:

1. **Browse** with `search_overview` to see what emails exist
2. **Select** specific thread IDs of interest
3. **Fetch details** using `fetch_email_bodies` for the selected threads
4. **Extract attachments** using `extract_attachment_by_filename` if needed

This approach minimizes context usage and API calls while still providing full functionality when needed.
