# Day 1: The "Secret Handshake" (TLS Fingerprinting)

**Objective:** Understand why your Python script gets blocked even when you use the correct User-Agent.
**Tools:** Python, Wireshark (optional), `requests`.

---

## 1. The Problem: "But I changed the User-Agent!"

You write a script:
```python
import requests
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36...'}
requests.get('https://g2.com', headers=headers)
```
**Result:** 403 Forbidden / Cloudflare Challenge.

**Why?**
Using `requests` makes you LOOK like a python script, regardless of the User-Agent string.
Cloudflare doesn't just read the text you send. It looks at **HOW** you connect.

## 2. What is a TLS Fingerprint?

When your computer connects to a secure HTTPS site, it performs a "TLS Handshake" (Introduction).
During this handshake, your client sends a "Client Hello" packet containing:
1.  **Ciphers:** Which encryption methods do I support? (AES-256, ChaCha20, etc.)
2.  **Extensions:** What extra features do I support? (SNI, ALPN, Supported Versions).
3.  **Order:** The **order** in which you list these things matters.

**Chrome's Handshake:**
*   Starts with `GREASE` (random noise).
*   Lists 15 ciphers in a specific order.
*   Includes specific extensions like `status_request`.

**Python `requests` Handshake:**
*   Lists different ciphers.
*   Different order.
*   Missing browser-specific extensions.

**The Anti-Bot (JA3):**
Cloudflare creates a messy hash (signature) of your handshake called **JA3**.
*   Chrome's JA3 = `cd08e31494f9531f560d64c695473da9` (Trusted)
*   Python Requests JA3 = `python-requests-hash` (BLOCKED)

## 3. The Solution: Impersonation

You cannot fix this with `requests`. The underlying library (`urllib3` / `OpenSSL`) controls the handshake, and you can't easily change it in Python.

To pass, you must use a library that wraps a **Real Browser's SSL Library**.
We will use **`curl_cffi`** (Python binding for curl-impersonate).

## 4. Homework (Do this now)

1.  **Install `curl_cffi`:**
    ```bash
    pip install curl_cffi
    ```
2.  **Run this test script:**

```python
from curl_cffi import requests

# 1. Normal Request (Will likely fail or show Python fingerprint)
# Note: tlspy.org or tools.scrapfly.io/api/fp/ja3 shows your fingerprint
print("--- Normal Request ---")
try:
    r = requests.get("https://tools.scrapfly.io/api/fp/ja3")
    print(r.json().get('ja3_hash'))
except:
    print("Failed")

# 2. Impersonated Request (Mimics Chrome 110)
print("\n--- Impersonated Request ---")
r = requests.get(
    "https://tools.scrapfly.io/api/fp/ja3",
    impersonate="chrome110"
)
print(f"Status: {r.status_code}")
print(f"Your Fingerprint mimics: {r.json().get('browser_user_agent')}")
```

3.  **Compare:**
    Go to [tls.peet.ws](https://tls.peet.ws) in your real Chrome browser. Look at the JA3 hash.
    It should match (or be very close to) what your `curl_cffi` script produces.

---

**Next Up (Day 2):** We will build a robust scraper class using this technology to bypass a real target.
