# Day 16: The Signature (Handling Signed Requests)

**Objective:** Replicate "Signed" headers in Python.
**Concept:** HMAC / Message Signing.

---

## 1. The Symptom
You copy the cURL from Day 15. It works.
You wait 5 minutes. You run it again. **403 Forbidden.**
Why?
Look at the headers again.
`X-Signature: 8a7b3c...`
OR
`X-Request-Id: 17099...`

These headers are generated using a **Timestamp**. If the timestamp in the header is old, the server rejects it.

## 2. The Solution
You must find the Logic that generates this string.
It is almost always: `Hash(Method + URL + Body + Timestamp + SecretKey)`.

## 3. Finding the Source
1.  Open Chrome DevTools -> **Sources** tab.
2.  Press `Ctrl+Shift+F` (Search All Files).
3.  Search for the Header Name (e.g., `"X-Signature"`).
4.  You will find a line like:
    ```javascript
    headers['X-Signature'] = generateSig(url, payload);
    ```
5.  Set a Breakpoint there.
6.  Refresh the page.
7.  Step into the function (`F11`) to see how it works.

## 4. Porting to Python
Once you understand the math (e.g., "It takes the current time in seconds, appends 'MY_SECRET', and MD5 hashes it"), you write it in Python.

```python
import time
import hashlib

def generate_signature(url):
    timestamp = str(int(time.time()))
    secret = "MY_Secret_Key_Found_In_JS"
    
    raw = url + timestamp + secret
    sig = hashlib.md5(raw.encode()).hexdigest()
    
    return {
        "X-Timestamp": timestamp,
        "X-Signature": sig
    }
```

## 5. Homework
1.  This is theoretical until you find a target.
2.  Look at **Shopee** or **Lazada** (Mobile APIs). They use this heavily.
3.  Try to find a site that has a dynamic header and find the JS code that builds it.
