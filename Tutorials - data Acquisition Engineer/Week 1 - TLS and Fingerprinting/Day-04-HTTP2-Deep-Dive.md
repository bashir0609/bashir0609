# Day 4: Speaking the Language (HTTP/2 vs HTTP/1.1)

**Objective:** Understand why "Header Order" gets you blocked.
**Concept:** HTTP/2 Multiplexing & Pseudo-Headers.

---

## 1. The "Protocol" Fingerprint
TLS (Day 1) covers the *Encryption* layer.
HTTP/2 covers the *Application* layer.
If you pass the TLS check but fail the HTTP/2 check, you still get blocked.

## 2. HTTP/1.1 (The Old Way)
*   **Format:** Plain Text.
*   **Headers:** Case-insensitive, usually sent in any order.
*   **Connection:** One request per connection (mostly).

## 3. HTTP/2 (The New Way)
*   **Format:** Binary Format.
*   **Pseudo-Headers:** You don't send `Host: google.com`. You send `:authority: google.com`.
    *   `:method`
    *   `:path`
    *   `:scheme`
    *   `:authority`
*   **Order:** Browsers send these Pseudo-Headers **FIRST**, in a strict order.
*   **Header Compression (HPACK):** Headers are compressed.

## 4. The Trap
If you use Python `requests`:
1.  It is **HTTP/1.1 ONLY**.
2.  If you connect to a site like `G2.com` which is Cloudflare protected...
3.  Cloudflare sees: "This client supports TLS 1.3 (Modern) but is using HTTP/1.1 (Ancient)."
4.  **Anomaly Detected -> BLOCK.**

Real browsers *always* prefer HTTP/2 over TLS 1.2+.
Mixed signals (Modern TLS + Ancient HTTP) = Bot.

## 5. Controlling HTTP/2 with curl_cffi
By default, `curl_cffi` tries to negotiate HTTP/2.
However, you must be careful with **Header Order**.

```python
# Bad Order (Mixed)
headers = {
    "User-Agent": "Chrome...",
    "Accept": "*/*",
    ":authority": "google.com" # WRONG! You can't set pseudo-headers manually in some libraries
}

# The Browser Order usually is:
# 1. :method
# 2. :authority
# 3. :scheme
# 4. :path
# 5. User-Agent
# 6. Accept...
```

`curl_cffi` handles the pseudo-headers automatically. But for the *normal* headers, you should try to match Chrome's order:
1.  `Host` (or Authority)
2.  `Connection`
3.  `Cache-Control`
4.  `sec-ch-ua`
5.  `User-Agent`
6.  `Accept`...

## 6. Homework
1.  Use `tshark` again.
2.  Look for "HTTP2" protocol in the output.
3.  Verify your script is actually sending `SETTINGS` frames (HTTP/2 handshake).
