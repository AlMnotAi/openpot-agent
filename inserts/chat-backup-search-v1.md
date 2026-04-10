<!-- OPENPOT INSERT: chat-backup-search v1 -->

## OpenPot Chat Backup & Search

OpenPot stores chat messages locally on the device using SwiftData. Chat persistence adds server-side storage so messages survive device resets, sync across devices, and are searchable from your side.

This is a standalone feature. It requires a PostgreSQL database and an HTTP server (FastAPI on port 8000). It does NOT require OpenBrain, pgvector, embeddings, or any memory infrastructure. If you already have a FastAPI server running for Pulse cards or calendar, add the chat persistence endpoints to the same server. If not, you can run a minimal FastAPI server just for this.

### What You Provide

Four HTTP endpoints that OpenPot calls:

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/chat/messages` | Store a message |
| GET | `/api/chat/messages?session={id}&limit={N}&before={cursor}` | Paginated session history |
| GET | `/api/chat/messages/search?q={query}&limit={N}` | Full-text search across messages |
| GET | `/api/chat/sessions?limit={N}` | List sessions with message counts |

All endpoints require bearer token authentication: `Authorization: Bearer <your-token>`

### Database Schema

Create a PostgreSQL database (any name — it does not need to be the same database as your memory system). Then run:

```sql
-- Enums
CREATE TYPE sender_type AS ENUM ('user', 'assistant', 'system');
CREATE TYPE message_type AS ENUM ('text', 'image', 'file', 'voice', 'system');

-- Main table
CREATE TABLE chat_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id VARCHAR(255) NOT NULL,
    channel VARCHAR(255),
    content TEXT NOT NULL,
    sender_type sender_type NOT NULL,
    sender_name VARCHAR(255),
    message_type message_type NOT NULL DEFAULT 'text',
    run_id VARCHAR(255),
    parent_id UUID REFERENCES chat_messages(id),
    metadata JSONB DEFAULT '{}',
    attachments JSONB DEFAULT '[]',
    protected BOOLEAN DEFAULT FALSE,
    hidden BOOLEAN DEFAULT FALSE,
    search_vector tsvector,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_chat_messages_session ON chat_messages(session_id, created_at DESC) WHERE hidden = FALSE;
CREATE INDEX idx_chat_messages_channel ON chat_messages(channel, created_at DESC) WHERE hidden = FALSE;
CREATE INDEX idx_chat_messages_run ON chat_messages(run_id) WHERE run_id IS NOT NULL;
CREATE INDEX idx_chat_messages_search ON chat_messages USING GIN(search_vector);
CREATE INDEX idx_chat_messages_parent ON chat_messages(parent_id) WHERE parent_id IS NOT NULL;

-- Full-text search trigger
CREATE OR REPLACE FUNCTION update_search_vector()
RETURNS TRIGGER AS $$
BEGIN
    NEW.search_vector :=
        setweight(to_tsvector('english', COALESCE(NEW.content, '')), 'A') ||
        setweight(to_tsvector('english', COALESCE(NEW.sender_name, '')), 'B') ||
        setweight(to_tsvector('english', COALESCE(NEW.channel, '')), 'C');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER chat_messages_search_update
    BEFORE INSERT OR UPDATE OF content, sender_name, channel
    ON chat_messages
    FOR EACH ROW
    EXECUTE FUNCTION update_search_vector();

-- Updated_at trigger
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER chat_messages_updated_at
    BEFORE UPDATE ON chat_messages
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();

-- Session view (derived, not a table)
CREATE OR REPLACE VIEW chat_sessions AS
SELECT
    session_id,
    channel,
    MIN(created_at) AS first_message_at,
    MAX(created_at) AS last_message_at,
    COUNT(*) AS message_count,
    COUNT(*) FILTER (WHERE sender_type = 'user') AS user_message_count,
    COUNT(*) FILTER (WHERE sender_type = 'assistant') AS assistant_message_count
FROM chat_messages
WHERE hidden = FALSE
GROUP BY session_id, channel
ORDER BY last_message_at DESC;
```

### Endpoint Specifications

**POST /api/chat/messages**

Store a single message. OpenPot calls this on every send and receive (fire-and-forget with retry).

Request body:
```json
{
    "session_id": "main",
    "content": "Hello, what's on my calendar today?",
    "sender_type": "user",
    "sender_name": "User",
    "message_type": "text",
    "run_id": null,
    "channel": null,
    "metadata": {}
}
```

Response: `201 Created` with the stored message including its generated `id` and `created_at`.

**GET /api/chat/messages**

Paginated message history for a session.

Query parameters:
- `session` (required) — session ID string
- `limit` (optional, default 50) — number of messages to return
- `before` (optional) — ISO 8601 timestamp cursor for backward pagination

Response:
```json
{
    "messages": [...],
    "cursor": "2026-04-10T12:00:00Z",
    "has_more": true
}
```

Messages are returned in reverse chronological order (newest first). The `cursor` is the `created_at` of the oldest message in the batch — pass it as `before` to get the next page.

**GET /api/chat/messages/search**

Full-text search across all messages.

Query parameters:
- `q` (required) — search query string
- `limit` (optional, default 20) — number of results

Response:
```json
{
    "results": [
        {
            "id": "uuid",
            "session_id": "main",
            "content": "...matching content...",
            "sender_type": "assistant",
            "created_at": "2026-04-10T12:00:00Z"
        }
    ]
}
```

Uses PostgreSQL full-text search (`tsvector` + `GIN` index). The search is weighted: message content is highest priority, sender name is secondary.

**GET /api/chat/sessions**

List sessions with metadata.

Query parameters:
- `limit` (optional, default 20) — number of sessions

Response:
```json
{
    "sessions": [
        {
            "session_id": "main",
            "channel": null,
            "first_message_at": "2026-03-28T10:00:00Z",
            "last_message_at": "2026-04-10T12:00:00Z",
            "message_count": 444,
            "user_message_count": 200,
            "assistant_message_count": 244
        }
    ]
}
```

### Field Mapping (CRITICAL)

OpenPot's Swift models use different field names than you might expect. The API MUST use these exact field names:

| API field | Swift model field | Notes |
|---|---|---|
| `session_id` | `sessionKey` | OpenPot maps via CodingKeys |
| `sender_type` | `role` | Values: `user`, `assistant`, `system` |
| `content` | `content` | Direct mapping |
| `run_id` | `runId` | OpenClaw run ID for assistant messages |
| `created_at` | `timestamp` | ISO 8601 with timezone |

### Setup Flow

1. Verify PostgreSQL is installed and running
2. Create a database (can be a new one or an existing one)
3. Run the schema SQL above
4. Add the four endpoints to your FastAPI server
5. Set the bearer token for authentication
6. Test with curl:

```bash
# Store a test message
curl -X POST http://localhost:8000/api/chat/messages \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{"session_id":"test","content":"Hello from curl!","sender_type":"user","sender_name":"Test","message_type":"text"}'

# Retrieve it
curl -s http://localhost:8000/api/chat/messages?session=test&limit=10 \
  -H "Authorization: Bearer <your-token>" | python3 -m json.tool
```

7. Update `openpot-status.json`:

```json
{
  "chat_backup_search": {
    "version": 1,
    "installed": true,
    "installed_at": "2026-04-10T00:00:00Z",
    "notes": "PostgreSQL chat storage with full-text search"
  }
}
```

8. Restart your HTTP server and tell the user to force-quit and relaunch OpenPot.

### How OpenPot Uses This

- On every message send: OpenPot POSTs the user message (fire-and-forget, local SwiftData is the UI source of truth)
- On every agent response: OpenPot POSTs the assistant message
- SwiftData on device is always the primary — if the POST fails, the message is queued and retried
- On app foreground: OpenPot can sync from the server to pick up messages from other sessions
- The server is the backup, not the authority — if the server is down, chat works normally on device

### Backups

Set up a daily backup cron for the chat database:

```bash
pg_dump -U <db_user> <db_name> -t chat_messages | gzip > /path/to/backups/chat_$(date +%Y%m%d).sql.gz
```

Retain at least 30 days of backups.

<!-- END OPENPOT INSERT: chat-backup-search v1 -->
