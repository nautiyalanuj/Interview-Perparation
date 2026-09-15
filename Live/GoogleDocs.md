
# How google doc works?
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

