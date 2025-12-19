# Day 3: The Chameleon (Implementation)

**Objective:** Write Python code that looks like Chrome.
**Library:** `curl_cffi` (Best maintained) or `tls_client` (Go wrapper).

---

## 1. Why curl_cffi?
Standard `requests` uses Python's OpenSSL.
`curl_cffi` uses a custom-compiled version of `curl` (the C library) that has been patched to support "Impersonation".
It can re-order ciphers and extensions to match specific browsers.

## 2. Installation
```bash
pip install curl_cffi
```

## 3. The Script
Create `tls_request.py`:

```python
from curl_cffi import requests

# 1. The Target
# We use a fingerprinting API to verify our "Costume"
url = "https://tls.peet.ws/api/all"

print("--- 🐢 Standard Python Requests (Authentication Failed) ---")
# This represents what you are doing now
try:
    # We use the standard requests library to simulate failure (pseudo-code)
    # import requests as standard_requests
    # print(standard_requests.get(url).json()['ja3_hash'])
    pass
except:
    pass

print("\n--- 🐇 Impersonated Chrome 120 (Authentication Success) ---")

# 2. The Impersonation
# impersonate="chrome120" tells the C library to use Chrome's exact SSL handshake
response = requests.get(
    url,
    impersonate="chrome120",
    headers={
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
    }
)

data = response.json()

# 3. Verification
print(f"Status Code: {response.status_code}")
print(f"JA3 Hash: {data['ja3_hash']}")
print(f"Akamai HTTP/2 Fingerprint: {data['akamai_hash']}")

# Analysis
# If successful, this JA3 hash will be identical to your Real Browser's hash.
# Anti-bots see this and think "Ah, a real user."
```

## 4. Troubleshooting
*   **Certificate Errors:** Sometimes local proxies mess up the certificate. Use `verify=False` only for debugging.
*   **Version mismatch:** If you use `impersonate="chrome100"` but send a `User-Agent: Chrome/120`, sophisticated bots (Datadome) will detect the mismatch. **Always match the version.**

## 5. Homework
1.  Run the script.
2.  Change `impersonate="safari15_3"`.
3.  Observe how the `ja3_hash` changes completely.
