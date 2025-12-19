# Day 24: Breaking the Seal (Bypassing SSL Pinning)

**Objective:** Force an app to trust your MITMProxy certificate even if it's programmed not to.
**Tools:** Frida, Objection.

---

## 1. The Problem: SSL Pinning
In Day 22, we installed a User Certificate. Browsers trust this.
**Apps do not.**
Modern Android Apps utilize "SSL Pinning". They have a list of valid certificates hardcoded inside the `.apk`.
When the app connects, it sees your MITMProxy certificate, compares it to the hardcoded list, sees no match, and kills the connection.
*Result:* "No Internet Connection" error in the app, and nothing in MITMProxy.

## 2. The Solution: Code Injection (Frida)
We cannot change the server. We cannot easily change the APK static code.
But we can change the **Running Memory**.

**Frida** allows us to inject a JavaScript snippet into the app process that rewrites the "fail" function to "success".
*Before:* `if (cert != trusted) return fail;`
*After:* `if (cert != trusted) return true; // Hacked by Frida`

## 3. Install Frida (PC & Phone)

**A. On PC:**
```bash
pip install frida-tools
pip install objection
```

**B. On Phone (Server):**
1.  You need a **Rooted** emulator (Genymotion is rooted by default).
2.  Download the `frida-server` binary for android-x86 (if emulator) or android-arm (if real phone) from GitHub.
3.  Push it to the phone:
    ```bash
    adb push frida-server /data/local/tmp/
    adb shell "chmod 755 /data/local/tmp/frida-server"
    adb shell "/data/local/tmp/frida-server &"
    ```

## 4. Bypassing with Objection (The Easy Way)
Objection is a wrapper around Frida that has pre-made scripts.

1.  **Find the Package Name:**
    Run the app on the phone.
    Type on PC: `frida-ps -U` (Lists running processes).
    Find execution name (e.g., `com.mcdonalds.app`).

2.  **Inject:**
    ```bash
    objection -g com.mcdonalds.app explore
    ```
    (This connects to the app).

3.  **Disable Pinning:**
    Inside the objection shell, type:
    ```bash
    android sslpinning disable
    ```

4.  **Test:**
    Refresh the app. It should now load, and you should see traffic in MITMProxy!

## 5. Homework
1.  Download a secure app (like Twitter or Yelp).
2.  Try to capture traffic. It will fail.
3.  Use Objection to disable pinning.
4.  Capture traffic. Success?
