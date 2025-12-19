# Day 1: The "Secret Handshake" (JA3 Fingerprinting)

**Objective:** Understand why you get blocked even with a valid User-Agent.
**Concept:** TLS Fingerprinting (JA3 / JA3S).

---

## 1. The Limitation of "User-Agent"
For 10 years, scrapers just changed `User-Agent: Mozilla/5.0...` and it worked.
Today, that header is ignored by advanced anti-bots (Cloudflare, Akamai, Datadome).
They look at **HOW** you connect, not just what you say you are.

## 2. What is a TLS Handshake?
When you connect to `https://google.com`, a complex negotiation happens **before** any HTTP data is sent.

**The Client Hello:**
Your browser sends a packet saying:
1.  **"I support these Encryption Ciphers"** (AES-128, ChaCha20, etc).
2.  **"I support these Extensions"** (SNI, ALPN, SessionTicket).
3.  **"I support these Elliptic Curves"** (x25519, secp256r1).

**The Fingerprint:**
*   **Chrome** always sends ciphers in a specific order.
*   **Firefox** sends them in a different order.
*   **Python `requests`** (via OpenSSL) sends a very minimal list, in a completely different order.

Cloudflare takes this list, hashes it, and creates a **JA3 Hash**.
If you claim to be "Chrome" (User-Agent) but your JA3 Hash matches "Python Script", you are **BLOCKED**.

## 3. Visualizing JA3
A JA3 string looks like this:
`771,4865-4866-4867-49195-49196-52393-49162,0-23-65281-10-11-35-16-5-13-18-51-45-43-27-21,29-23-24,0`

*   `771`: TLS Version
*   `4865-4866...`: Accepted Ciphers (Decimal IDs)
*   `0-23-65281...`: Extensions
*   `29-23-24`: Elliptic Curves

## 4. Why `requests` fails
The `requests` library uses the OS's OpenSSL. It doesn't have control over the *order* of ciphers.
To bypass this, we need a library that **Impersonates** a browser's SSL handshake.

## 5. Homework
1.  Open Chrome. Go to [tls.peet.ws](https://tls.peet.ws).
2.  Look at your **JA3 Hash**.
3.  Run this Python script:
    ```python
    import requests
    print(requests.get("https://tls.peet.ws/api/clean").json()['ja3_hash'])
    ```
4.  Compare the two hashes. They are different. That is why you get blocked.
