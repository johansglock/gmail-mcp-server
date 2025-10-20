# Token Usage Optimization Guide

## The Problem

Large email threads can consume 10k+ tokens when reading full content, filling up context quickly.

## The Solution

Use **progressive loading** and **summary mode** to reduce token usage by ~80%.

## Quick Reference

| Tool | Mode | Token Usage | Use When |
|------|------|-------------|----------|
| `search_overview` | - | ~100-500 | Finding threads |
| `get_thread_messages` | `summary_only: true` | ~1-2k | Quick thread overview |
| `get_thread_messages` | default | ~3-6k | Read full conversation |
| `get_thread_messages` | `max_body_length: 500` | ~2-3k | Brief content needed |
| `get_message` | - | ~1-2k | Read one specific message |
| `fetch_email_bodies` | - | ~2-4k | Legacy (first message only) |

## Optimization Strategies

### Strategy 1: Summary First (Recommended)

```javascript
// Step 1: Get lightweight summary (~1-2k tokens)
get_thread_messages(threadId, {summary_only: true})

// Step 2: Review snippets, identify messages of interest

// Step 3: Fetch specific messages (~1-2k each)
get_message(relevantMessageId)

// Total: ~2-4k tokens instead of ~11k
```

### Strategy 2: Controlled Body Length

```javascript
// Get full thread but limit body length
get_thread_messages(threadId, {
  summary_only: false,
  max_body_length: 1000  // ~2-4k tokens total
})
```

### Strategy 3: Targeted Reading

```javascript
// If you know which message you need:
get_message(specificMessageId)  // ~1-2k tokens

// Don't load the entire thread unnecessarily
```

## Real-World Examples

### Example 1: Read Latest Reply

**Old way (11k tokens):**
```javascript
get_thread_messages(threadId)  // Gets everything
// Read through all 5 messages to find the latest
```

**New way (2-4k tokens):**
```javascript
// 1. Get summary to see structure
get_thread_messages(threadId, {summary_only: true})  // ~1-2k

// 2. Last message is what we need
get_message(lastMessageId)  // ~1-2k

// Saved: ~7k tokens!
```

### Example 2: Scan Multiple Threads

**Old way (55k tokens):**
```javascript
// Get full content for 5 threads
for (threadId in threadIds) {
  get_thread_messages(threadId)  // ~11k each = 55k total
}
```

**New way (5-10k tokens):**
```javascript
// Get summaries for 5 threads
for (threadId in threadIds) {
  get_thread_messages(threadId, {summary_only: true})  // ~1-2k each = 5-10k total
}

// Then deep-dive into only the interesting ones
get_thread_messages(interestingThreadId)  // Add ~3-6k for one thread
```

### Example 3: Email Monitoring

**Task:** Check for important replies in 10 threads

**Efficient approach:**
```javascript
// 1. Get all thread summaries (10-20k tokens)
for (thread in threads) {
  summary = get_thread_messages(thread, {summary_only: true})

  // 2. Check if there are new messages
  if (summary.messageCount > lastKnownCount) {
    // 3. Get only the new message (1-2k tokens)
    newMessage = get_message(summary.messages.last.messageId)
  }
}

// Total: 10-25k tokens vs 110k for full content
```

## Field-by-Field Token Cost

### Summary Mode Fields (Minimal)
- `messageId`: ~20 chars
- `subject`: ~50 chars
- `from`, `to`: ~30 chars each
- `dateTime`: ~25 chars
- `snippet`: ~150 chars
- **Total per message: ~300 chars (~75 tokens)**

### Full Mode Fields (Comprehensive)
- All summary fields: ~300 chars
- `fullBody`: 500-8000 chars (default 2000)
- `labelIds`: ~50 chars
- `date`, `cc`, `inReplyTo`, etc.: ~100 chars
- `attachments`: ~100-500 chars
- **Total per message: 1000-9000 chars (~250-2250 tokens)**

## Best Practices

1. **Always start with summary_only** unless you know you need full content
2. **Use search_overview first** to find relevant threads cheaply
3. **Fetch individual messages** when you know exactly what you need
4. **Set max_body_length** to 500-1000 for quick reads, 2000 for normal use
5. **Only use 8000** when you absolutely need complete content
6. **Monitor token usage** - if you hit limits frequently, lower max_body_length

## Decision Tree

```
Need to read emails?
├─ Just checking what's in the thread?
│  └─ Use: get_thread_messages(summary_only: true)  [~1-2k tokens]
│
├─ Need to read one specific message?
│  └─ Use: get_message(messageId)  [~1-2k tokens]
│
├─ Need to read full conversation?
│  ├─ Short emails expected?
│  │  └─ Use: get_thread_messages(max_body_length: 1000)  [~2-4k tokens]
│  │
│  └─ Normal length?
│     └─ Use: get_thread_messages() (default)  [~3-6k tokens]
│
└─ Need absolutely everything?
   └─ Use: get_thread_messages(max_body_length: 8000)  [~8-15k tokens]
```

## Measuring Your Usage

The AI will warn you when responses are large:
- "~1k tokens" - Very efficient ✅
- "~3k tokens" - Good balance ✅
- "~6k tokens" - Acceptable for important reads ⚠️
- "~11k tokens" - Too large, consider optimization ❌

## Migration Guide

### If you're getting token warnings:

**Before:**
```javascript
get_thread_messages(threadId)  // Default used to be 8000 chars
```

**After:**
```javascript
// For overview:
get_thread_messages(threadId, {summary_only: true})

// For reading:
get_thread_messages(threadId)  // Now defaults to 2000 chars

// For specific messages:
get_message(messageId)  // Most efficient
```

## Summary

- **Summary mode**: 80% token savings
- **Reduced body length**: 50-70% token savings
- **Targeted fetching**: 90% token savings
- **Progressive loading**: Best of all worlds

Start small, load more only when needed!
