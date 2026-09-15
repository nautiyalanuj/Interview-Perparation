# Video Streaming
- I have gone through multiple site like youtube, hellointerview, netflix, aajtak, hotstart and xhamster to identify how each sites are streaming the videos. 
- I have captured the .har file for each and shared same with AI and thus sharing the analysis which I have found on basis of same.

---

### **1. Standard Public HLS / MPEG-DASH**

* **How It Operates:** Video manifests (`.m3u8` or `.mpd`) and media segments (`.ts`/`.mp4`) are served directly over open, unencrypted CDN edge nodes.
* **Sites Using It:** **xhamster** (Free VOD), **[AajTak](https://www.aajtak.in?utm_source=gemini)** (Free VOD / Live web streams).
* **Technical Details:**
* `xHamster` relies heavily on standard HLS (`hls.js` player reading `.m3u8` playlists and fetching `.av1`/`.h264` chunks).
* `AajTak` uses public HLS to stream breaking news clips freely across news portals and web browsers without needing logins or DRM licenses.



---

### **2. Proprietary Protocol / Custom Binary Streaming (SABR & UMP)**

* **How It Operates:** Instead of using standard plain-text HLS manifests (`.m3u8`), requests go directly to media servers using a binary envelope protocol.
* **Sites Using It:** **[YouTube](https://www.youtube.com?utm_source=gemini)** (Free & Premium VOD / Live).
* **Technical Details:**
* You are **correct** that YouTube does not use standard HLS for web playback. Historically, YouTube used standard **MPEG-DASH**. Modern YouTube uses a proprietary protocol known as **SABR** (Server-Adaptive Bitrate Streaming) over **UMP** (Unified Media Protocol).
* Requests hit endpoints like `[googlevideo.com/videoplayback](https://googlevideo.com/videoplayback)?...`. Instead of returning static `.m3u8` lists, YouTube’s backend pushes binary Protobuf chunks (`googlevideo/ump`) directly to the client player. This allows YouTube to optimize adaptive bitrate switching, inject dynamic ad insertions, and prevent easy video scraping.



---

### **3. Token / Signed-URL Authentication**

* **How It Operates:** The player authenticates against an API server to get a short-lived, cryptographically signed URL containing expiration timestamps and query tokens.
* **Sites Using It:** **[HelloInterview](https://www.hellointerview.com?utm_source=gemini)** (Premium/Private content).
* **Technical Details:**
* `HelloInterview` hosts paid coding/system design courses. Videos are protected via Cloudflare Stream, AWS CloudFront, or Wistia using signed tokens. Non-authenticated users get a `403 Forbidden` from the CDN edge if they attempt to load the underlying video manifest.
* Paywalled VOD on tube sites uses session-bound signed URLs to prevent direct hotlinking on third-party sites.



---

### **4. DRM-Protected HLS/DASH (Widevine, FairPlay, PlayReady)**

* **How It Operates:** Video frames are encrypted using AES-128. Before playback begins, the browser's native **Content Decryption Module (CDM)** issues a key challenge to a DRM License Server.
* **Sites Using It:** **[Netflix](https://www.netflix.com?utm_source=gemini)**, **[Disney+ Hotstar](https://www.hotstar.com?utm_source=gemini)**.
* **Technical Details:**
* Both `Netflix` and `Disney+ Hotstar` encrypt 100% of their premium catalog using MPEG-DASH (Widevine on Chrome/Android) and HLS (FairPlay on Safari/iOS).
* Even if you open DevTools and download the `.mp4` / `.m4s` segment, it remains encrypted garbage without an authenticated decryption key processed inside your device's hardware enclave (TEE).



---

### **5. Low-Latency HLS (LL-HLS) / WebRTC**

* **How It Operates:** Sub-second (WebRTC) or ultra-low latency (~2 sec via LL-HLS) transport for real-time interactivity.
* **Sites Using It:** (WebRTC), **[Disney+ Hotstar](https://www.hotstar.com?utm_source=gemini)** (LL-HLS for Live Sports).
* **Technical Details:**
* **LL-HLS:** Used by `Hotstar` during massive live events (e.g., IPL Cricket matches with tens of millions of concurrent viewers) to reduce latency down to 2–3 seconds while maintaining high CDN scalability.



---

### Protocol Matrix Across Your Examples

| Site | Primary Delivery Method | Protocol / Format | Security Model |
| --- | --- | --- | --- |
| **YouTube** | **Proprietary Adaptive Streaming** | UMP / SABR over `googlevideo.com` (Binary Protobuf) | Signed Playback Tokens + Client JS obfuscation |
| **HelloInterview** | **Token-Auth Stream** | HLS / DASH via Cloudflare/Wistia | Signed CDN URLs (Session Auth) |
| **Netflix** | **Full DRM** | MPEG-DASH / HLS | DRM License Server (Widevine/FairPlay) |
| **AajTak** | **Public HLS** | Standard HLS (`.m3u8`) | Open / Public CDN |
| **Disney+ Hotstar** | **DRM + LL-HLS** | MPEG-DASH / LL-HLS | DRM License + Signed Tokens |
| **xHamster** | **Standard HLS** | HLS (VOD) / WebRTC (Live Cams) | Public CDN (Free VOD) / WebSockets (Cams) |


