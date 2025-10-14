## Resumable Uploads/Downloads
**Resumable Uploads/Downloads** enabled a partially transferred file to be resumed from where it left off, without restarting the transfer from the beginning. This is critical for large files, unstable network environments, mobile use-cases, or when ensuring data integrity and saving bandwidth.

**Why it matters in Mobile**
- Network interruptions are common on mobile (eg. user switches from WiFi to mobile data)
- Users may background the app mid-transfer
- Resuming avoids starting over, improving user experience and reducing data costs.

**Concept**:
1. **Chunked Uploads**
	- Uploads are broken into fixed-size chunks (eg. 1MB)
	- Each chunk is uploaded separately with an index or byte-range
	- Server stores which chunks are received and responds accordingly
2. **Byte Range Requests**
	- Downloaders use the `Range` header to specify the byte positions: "`Range: bytes=100000-`"
	- Server responds with: "`HTTP/1.1 206 Partial Content`"
3. **Upload Session ID / Token**
	- Initiate an upload session
	- Server returns a session ID, stored by the client
	- All subsequent chunks uploads reference this session
	- Uploads can resume even after app restarts if session ID is retained
4. **ETag or Last-Modified**
	- For resumable downloads, clients can cache `ETag` or `Last-Modified` header
	- They revalidate before resuming or restarting

---
### iOS: URLSession & Background Uploads
- **URLSessionUploadTask** with background configuration
- Resumable downloads are **native** via `downloadTask(withResumableData:)`
- Uploads need custom chunk logic

```swift
// Resumable download
let task = session.downloadTask(withResumeData: resumeData)
```

---
### Android: WorkManager / OkHttp
- **OkHttp** supports resuming downloads with `Range` headers
- Uploads require **manual chunking logic** or use of libraries like **Tus**, **Firebase**, or **Multipart upload frameworks**
- **WorkManager** can schedule resilient background tasks

---

**Design Considerations**:
- Small uploads (<10 MB): Regular upload
- Large uploads/downloads: Use chunked transfers
- Intermittent connections: Background + resumable + retry
- High-concurrency needs: Keep track of per-session offset

**Example - Google Drive Resumable Upload**:
1. Client sends `POST` to initiate session
2. Server returns an upload URL (with session token)
3. Client uploads chunks using `PUT` with `Content-Range`
4. If interrupted, resend last chunk