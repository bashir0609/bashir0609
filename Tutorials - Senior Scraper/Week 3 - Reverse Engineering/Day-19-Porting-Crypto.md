# Day 19: The Alchemist (Porting Crypto)

**Objective:** Translate a JS function like `a = md5(b + "salt")` into Python `hashlib.md5(b + "salt")` to generate valid API signatures.
**Why?** If you just copy the header from DevTools, it expires in 5 minutes. If you generate it yourself, you run forever.

---

## 1. Identify the Algorithm
In Day 17, you found a function `generateSignature(payload)`.
It usually looks like this:
```javascript
// Minified JS
function s(t) {
    var e = "MY_SECRET_SALT_888";
    return crypto.sha256(t + e).toString();
}
```

You must identify:
1.  **Input:** What is `t`? (Usually the URL or JSON body).
2.  **Secret:** What is `e`? (A hardcoded string in the JS).
3.  **Algo:** SHA256? MD5? HMAC-SHA1?

## 2. Common Obfuscation
They won't name it `crypto.sha256`. It will look like math:
`v[0] = (v[0] >>> 2) ^ k[3]...`
This is "Bit/Shift operations".
**Pro Tip:** Copy the code into ChatGPT/Claude and ask: *"What hashing algorithm is this JS implementation?"*
It will usually tell you *"This looks like a standard MD5 implementation."*

## 3. The Python Port
Once you know the logic (Input + Secret + Algo), rewrite it.

**JS Code:**
```javascript
// Found in bundle.js
x = Math.floor(Date.now() / 1000); // Timestamp
y = "deadbeef"; // Static Key
sig = md5(x + y);
```

**Python Equivalent:**
```python
import hashlib
import time

def generate_signature():
    # 1. Match the timestamp logic EXACTLY (seconds vs milliseconds?)
    timestamp = int(time.time()) 
    
    # 2. The static key
    secret = "deadbeef"
    
    # 3. Concatenate
    raw_string = f"{timestamp}{secret}"
    
    # 4. Hash (Encode to bytes first!)
    signature = hashlib.md5(raw_string.encode('utf-8')).hexdigest()
    
    return signature, timestamp
```

## 4. Testing
1.  Get a valid signature/timestamp pair from your Browser Network Tab.
2.  Plug that specific timestamp into your Python function.
3.  Does your Python `signature` match the Browser `signature`?
    *   **Yes:** You cracked it.
    *   **No:** You are missing a variable (maybe User-Agent is included in the hash?).

## 5. Homework
1.  This is hard to find a public example for that stays static.
2.  Try `Shopee` or `Lazada` (E-commerce). They use heavy signing on their search API.
3.  Find the header `If-None-Match-*` or similar.
