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

## Comparison with `search_threads`

| Feature | `search_overview` | `search_threads` |
|---------|-------------------|------------------|
| Thread ID | ✅ | ✅ |
| Subject | ✅ | ✅ |
| Last Message ID | ✅ | ❌ |
| Last Message Date/Time | ✅ | ❌ |
| Sender (From) | ❌ | ✅ |
| Snippet | ❌ | ✅ |
| Message Count | ❌ | ✅ |
| Attachments | ❌ | ✅ |
| Drafts | ❌ | ✅ |
| Context Usage | Minimal | Higher |
| Performance | Faster | Slower |

## When to Use

**Use `search_overview` when:**
- You want to quickly see what emails exist
- You need to browse many emails efficiently
- You only need basic metadata to decide which emails to investigate further
- You want to minimize context token usage

**Use `search_threads` when:**
- You need to see email snippets/previews
- You want to know about attachments
- You need to see existing drafts
- You want complete thread information

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
