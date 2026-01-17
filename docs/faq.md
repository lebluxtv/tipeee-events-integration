# FAQ

### Is this an official Tipeee API?
No. This documentation describes **observed behavior** of the Tipeee frontend.
It is not officially supported or documented by Tipeee.

### Is this related to TipeeeStream?
No. TipeeeStream is a separate product with a different event system.

### Does this use polling?
No. Events are delivered via a **persistent Socket.IO connection** (push model).

### Is the payload stable?
No. The payload is not versioned and may change without notice.

### Why document this?
To enable clean, event-driven integrations without scraping or polling.
