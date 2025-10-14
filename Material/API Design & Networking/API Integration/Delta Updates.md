## Delta Updates
When a client app needs to sync data with a backend, **delta updates** help fetch *only what has changed*, not the entire dataset. This conserves bandwidth, improves performance, and enhances UX in mobile environments. Here are the three major techniques:

---
### Timestamps
- Server maintains a `last_updated_at` timestamp on each record
- Client stores the last time it synced (`lastSyncedAt`) and asks for "data updated after X"

**Example**:
```http
GET /messages?updated_since=2024-07-01T10:00:00Z
```

**Pros**:
- Simple and widely supported
- Easy to implement both on client and server

**Cons**:
- Requires synchronized clocks (client/server)
- Risk of missing updates if server time changes or network latency causes a race
- Duplicate data if multiple updates occur with the same timestamp

---
### Sequence ID / Incrementing ID
- Server assigns monotonically increasing ID to each change (eg. `change_id` or `version_id`)
- Client keeps track of the last seen `change_id` and requests updates since then

**Example**:
```http
GET /events?since_id=5012
```

**Pros**:
- No clock sync required
- Avoids duplicate records more cleanly than timestamps
- Good for ordered and sequential systems (feeds, logs, events)

**Cons**:
- More complex server logic required
- Can't represent deleted items unless tombstones are used

---
### ETag
- Server sends an `ETag` header (a hash or version string) for a resource
- Client stores this ETag and uses `If-None-Match` in subsequent requests
- If the data hasn't changed, server returns HTTP `304 Not Modified`

**Flow**:
```http
Client:
GET /profile
→ Server: 200 OK, ETag: "abc123"

Client:
GET /profile
If-None-Match: "abc123"
→ Server: 304 Not Modified
```

**Pros**:
- Fully integrated into HTTP spec
- No need to implement custom fields for timestamps or IDs
- Efficient for caching full objects (not suitable for partial updates)

**Cons**:
- Doesn't scale well for large collections
- Works best for single resource GETs, not list APIs

---

| Technique       | Clock Sync | Granularity  | Detect Deletes     | Complexity | Best For                  |
| --------------- | ---------- | ------------ | ------------------ | ---------- | ------------------------- |
| **Timestamp**   | ❌ Needed   | Seconds/ms   | ❌ No               | ✅ Simple   | General lists, basic sync |
| **Sequence ID** | ❌ No       | Precise      | ❌ Needs tombstones | 🔶 Medium  | Feeds, logs               |
| **ETag**        | ❌ No       | Whole object | ✅ Yes (via 304)    | 🔶 Medium  | Single resource caching   |
