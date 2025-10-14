## Pagination
When dealing with large datasets in mobile apps (chat messages, product lists, notifications, etc), **pagination** prevents over-fetching and improves **performance, memory, and network efficiency**. There are 4 major strategies:

---
### Limit-Offset Pagination
Client sends `limit` (how many items) and `offset` (where to start)

**Example**:
```http
GET /posts?limit=20&offset=40
```

**Use Case**:
- Simple, intuitive
- Works well for **static datasets**

**Cons**:
- Inconsistent with **inserts/deletes**
- Expensive on large datasets (`offset` needs to scan and skip rows)
- **Prone to duplicates or missing items** if data mutates between requests

**When to Use**:
- Admin panels, reporting tools
- ~~Infinite scroll feeds, real-time lists~~

---
### Page-based Pagination
Client sends **page number** and size

**Example**:
```http
GET /posts?page=3&pageSize=20
```

**Use Case**:
- Works well when UI has **discrete pages** (eg. 1, 2, 3)
- Common in **e-commerce apps, dashboards**

**Cons**:
- Internally often same as offset-based (`offset = (page - 1) * size`)
- Suffers from same **inconsistency issues** on data mutation

**When to Use**:
- Static UIs, catalogs
- ~~Realtime timelines or chat~~

---
### Keyset Pagination
Paginate based on **unique, ordered key** (like `created_at` or `id`)

**Example**:
```http
GET /posts?limit=20&after=2024-07-10T12:00:00Z
```
The server returns *after* a certain value of the key.

**Use Case**:
- **Highly consistent**, fast, and scalable
- Avoids duplicate/missing items on inserts/deletes
- Uses index-friendly queries

**Cons**:
- Requires **sortable field** (eg. `id`, `timestamp`)
- Can't jump to arbitrary pages ("show page 9")

**When to Use**:
- **Feeds, chats, timelines**, infinite scroll
- ~~Static data that needs arbitrary access~~

---
### Cursor-based Pagination
Backend returns a **cursor token** (opaque), which the client passes back to request the next page

**Example:**
```http
GET /posts?limit=20&cursor=abc123xyz
```

**Use Case**:
- Great for **complex queries or composite keys**
- Can include sorting, filters, or encrypted data in the cursor

**Cons**:
- **Opaque** (can't bookmark/share pages)
- Requires **stateless server logic** to decode/generate cursors
- Hard to debug manually

**When to Use**:
- APIs with filters, multiple sort orders
- Highly dynamic datasets
- ~~Human-readable pagination needs~~

---

### Comparison

| Strategy     | Supports Arbitrary Jumps | Handles Inserts/Deletes Well | Cursor Friendly | Debuggable | Best For                   |
| ------------ | ------------------------ | ---------------------------- | --------------- | ---------- | -------------------------- |
| Limit-Offset | ✅                        | ❌                            | ❌               | ✅          | Simple, static data        |
| Page-Based   | ✅                        | ❌                            | ❌               | ✅          | Product listings, catalogs |
| Keyset       | ❌                        | ✅                            | Optinal         | Medium     | Chat, feeds, activity logs |
| Cursor-Based | ❌                        | ✅                            | ✅               | ❌          | Infinite scroll, APIs      |

### Mobile-Specific Advice

| Concern         | Recommendation                                  |
| --------------- | ----------------------------------------------- |
| Pull-to-refresh | Use Keyset or Cursor (based on newest item key) |
| Infinite scroll | Keyset/Cursor only                              |
| Background sync | Avoid Offset, use timestamps                    |
| Offline caching | Store last seen key/cursor instead of offset    |
