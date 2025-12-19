# Day 26: The Vault (Hard App Interception)

**Objective:** Scrape an app that uses Request Signing.
**Target:** Food Delivery (UberEats/DoorDash) or E-commerce (Shopee).

---

## 1. The Symptom
You intercept the JSON (Day 25).
You copy headers to Python.
**It fails (403).**
You look at the headers:
`X-Signature: a1b2c3d4...`
`X-Timestamp: 1234567890`

## 2. The Solution (Decompiling)
To act like the App, we must **Think** like the App. We need to find the code that generates `X-Signature`.

## 3. Tools
*   **JADX-GUI:** The best free APK decompiler.
*   **APKMirror:** Where you download the `.apk` file (since you can't easily get it from the phone).

## 4. The Workflow
1.  Download the Target APK (e.g., `com.target.apk`) from APKMirror.
2.  Open it in **JADX-GUI**.
3.  Click "Text Search" (Top left).
4.  Search for `"X-Signature"` (The header name).
5.  You will find a Java function:
    ```java
    public String signRequest(String url, String body) {
        String secret = "MY_Secret_Key_v2";
        return HMAC(url + body, secret);
    }
    ```
6.  **Translate this Logic to Python.** (See Week 3 Day 16).

## 5. What if it's Native Library (`.so`)?
Sometimes the logic is hidden in C++ code (`libnative.so`).
JADX cannot read C++.
**Solution:**
1.  **Frida Hooking:** Don't read the code. Just use Frida to **Call the function** on the phone.
2.  *Advanced:* "RPC" (Remote Procedure Call). Your Python script asks the Phone "Please sign this URL", and the Phone replies with the signature.

## 6. Homework
1.  Download JADX-GUI.
2.  Open any APK you have.
3.  Search for "API_KEY" or "Authorization".
4.  See how readable the code is. (It's usually very readable).
