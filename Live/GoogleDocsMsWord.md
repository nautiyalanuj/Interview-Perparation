
# How google doc works?
- Note google uses HTTP/3 for same.
- Google Docs actually relies on a hybrid model: it uses HTTP requests (XHR/Fetch) for outbound user actions, but relies on streaming HTTP connections (long-polling) rather than simple short-polling for real-time sync.
- **bind?...** (HTTP Long-Polling): This persistent request at the top is Google's proprietary channel (often called Channel API or BrowserChannel). Instead of standard WebSockets, Google uses a long-lived HTTP connection where the server holds the response open to stream incoming document edits from other users in real time.
- **save?...** (HTTP POST/Fetch): Triggered whenever you make changes (typing, formatting). Your local operational transformation (OT) diffs are sent to the server via standard HTTP requests.
  - Why are we using HTTP, is it not inefficient where we have to create full TCP connection every-time when we have to send message?
    - Browsers use HTTP Keep-Alive (HTTP/1.1) or connection multiplexing (HTTP/2 and HTTP/3). The underlying TCP connection and TLS session stay open in the background.
    - No Re-Handshaking: When the client sends an HTTP POST (like a save or assistwriting request) or initiates a new bind request, it reuses the already-open TCP connection. There is no penalty for opening new TCP connections or performing TLS negotiation every time. 
- **assistwriting?...**: API requests triggering AI/Smart Compose suggestions as you type.
- **log?...**: Analytics and client telemetry endpoint.
- Why Not WebSockets?
  - Google Docs originally predates widespread WebSocket support and built its real-time collaboration pipeline around HTTP long-polling and Server-Sent Events (SSE) architectures, which work reliably across strict corporate firewalls, proxies, and load balancers.
- Why multiple **bind** request ?
  -  We are seeing multiple bind requests because standard HTTP long-polling connections cannot stay open indefinitely.
  -  Intermediate infrastructure (proxies, load balancers, NAT gateways, and browser engines) automatically terminate HTTP connections if they stay open too long (usually every 30 to 60 seconds).
  -  Why we see noop Messages?
    -  noop (No Operation): These are keep-alive heartbeats sent by the server at regular intervals (e.g., every 15–30 seconds) while no actual document changes are occurring.
    -  Preventing Idle Drop: If the server remains completely silent for too long, network firewalls will assume the TCP connection is dead and silently drop it. The noop keeps the channel active until the timeout threshold is reached and a clean reconnect occurs.
  - Client Side (Outbound HTTP Requests + Keep-Alive):
    - When your browser sends an edit to Google Docs (e.g., typing a character), it sends an HTTP POST request. Instead of opening a brand-new TCP and TLS connection every single time, it reuses an already-established TCP connection using HTTP Keep-Alive. Reusing the connection eliminates the overhead of network round-trips for TCP three-way handshakes and TLS encryption negotiations. 
  - Server Side (Long Polling Stream):
    - The server needs a way to push real-time document updates to you when other users type. Standard HTTP requests are client-initiated only (the server cannot spontaneously reach out to your browser). To work around this without breaking standard web infrastructure, the server uses HTTP Long Polling (or Chunked Streaming). The client opens a bind request, and the server intentionally leaves it open, pushing small chunks of data whenever updates occur. 

<img width="424" height="279" alt="image" src="https://github.com/user-attachments/assets/7363d576-16ae-49a0-ac74-d7ab1fa6bb17" />
<img width="1414" height="484" alt="image" src="https://github.com/user-attachments/assets/e5cfa388-f6bc-4883-817b-c147cece8af9" />


# How ms word works?
- Again I have done this analysis based on har file and AI to debug same.

In Microsoft Office Online (Word Web / Cobalt sync engine), **`GetChanges`** and **`PutChanges`** are the core REST/HTTP API operations used to synchronize document content between the client browser and the server.

---

### 1. `PutChanges` (Outbound Sync)

* **What it does:** Sends document edits from the user's browser up to the server.
* **Mechanism:** When a user types, formats, or edits text, the client batches these modifications into **Operational Transformation (OT) deltas** or **Cobalt binary atoms**.
* **Payload Structure:** Inside the request body of a `PutChanges` call, you will find encoded binary streams or compressed payloads representing actions like *insert character*, *delete node*, or *update style*.
* **Verification:** A successful `PutChanges` request returns an HTTP `200 OK` status with confirmation metadata, acknowledging that the server accepted the edits and updated the document's central lineage revision.

---

### 2. `GetChanges` (Inbound Sync)

* **What it does:** Fetches latest edits made by other co-authors or catches up on missing changes.
* **Mechanism:** The browser passes its current document revision/sequence ID to the server. The server responds with all document changes made after that revision ID.
* **Role in Real-Time Editing:**
* While active typing notifications often arrive over WebSockets (`rtchub`), `GetChanges` acts as the **source of truth catching/reconciliation mechanism**.
* If a WebSocket frame drops, the connection reconnects, or a user opens a document that was edited offline, the browser issues a `GetChanges` call to download the exact stream of missing operations and bring the document up to date.



---

### Summary of Workflow

```
[User Types Text]
       │
       ├── (1) Fast UI Broadcast  ──>  WebSocket (rtchub) ──> Other Co-Authors
       │
       └── (2) Persistence & Sync ──>  HTTP POST PutChanges  ──> Server Storage
                                                                     │
[Other User Reconnects / Catches Up] <── HTTP POST GetChanges ───────┘

```

Together, `PutChanges` and `GetChanges` ensure that edits are safely persisted to OneDrive/SharePoint storage and that no document changes are lost even if the real-time WebSocket connection temporarily drops.


# Google Doc vs Ms Word
### **Google Docs Architecture (HTTP POST + HTTP Streaming / Long Polling)**

#### **Advantages**
* **High Firewall and Corporate Proxy Compatibility**: Standard HTTPS POST requests and HTTP long-polling (`bind`) pass effortlessly through corporate firewalls, proxies, and middleboxes that frequently inspect or block custom socket protocols.
* **Stateless Edge Load Balancing**: Outbound write requests (`save`) can be statelessly routed to any available edge worker or app server without requiring sticky socket sessions.
* **Explicit Write Acknowledgments**: Sending edits via HTTP POST returns immediate HTTP status codes (like `200 OK`) to confirm the server successfully acknowledged the Operational Transformation (OT) delta.
* **Multiplexed Parallel Transport**: Running over HTTP/2 or HTTP/3 (QUIC) allows outbound save requests and inbound `/bind` streaming requests to operate simultaneously in parallel over a single connection. Under HTTP/3, packet drops on one stream do not block other active streams.
* **Simpler State Recovery**: Connection state is statelessly recoverable via sequence IDs, avoiding the complex reconnection state management required by persistent WebSockets.

#### **Disadvantages**
* **Periodic Reconnection Cycles**: Long-polling streams cannot stay open indefinitely due to proxy, NAT gateway, and load balancer timeout limits (typically terminating every 30–60 seconds), requiring the browser to continuously initiate new `/bind` requests.
* **Heartbeat Overhead**: The server must periodically send `noop` (no-operation) keep-alive messages over idle streams to prevent middlebox firewalls from silently dropping the TCP/UDP connection.

---

### **Microsoft Word Web Architecture (Hybrid: WebSockets + HTTP REST APIs)**

#### **Advantages**
* **Low-Latency Signaling & Presence**: Uses WebSockets (`rtchub` via ASP.NET Core SignalR) as a dedicated push channel for live cursor tracking, user presence, and instant typing notifications.
* **Transactional Storage Persistence**: Uses dedicated HTTP REST APIs (`PutChanges`) to commit edits, providing fail-safe HTTP acknowledgments when persisting changes directly to OneDrive/SharePoint storage.
* **Bandwidth Efficiency**: Broadcasts lightweight signaling events over WebSockets (e.g., notifying clients that new changes exist) rather than constantly piping heavy document payloads over the socket connection.
* **Resilient Catch-Up Engine**: If the WebSocket connection drops, no document edits are lost because the client fetches missing changes via `GetChanges` REST calls upon reconnecting.

#### **Disadvantages**
* **Dual Infrastructure Complexity**: Maintaining a dedicated WebSocket signaling network alongside an HTTP REST API storage layer increases client and server operational complexity.
* **Strict Firewall Sensitivity**: Corporate environments that block WebSocket connection upgrades (HTTP Status `101 Switching Protocols`) force the application to fall back to HTTP long-polling or Server-Sent Events (SSE).
* **Reconnection Overhead**: Re-establishing dropped WebSocket connections requires managing socket session state alongside sequence IDs and catching up via HTTP endpoints.

---

💡 Would you like to analyze how both platforms handle offline edit queues and sync reconciliation upon reconnecting?
