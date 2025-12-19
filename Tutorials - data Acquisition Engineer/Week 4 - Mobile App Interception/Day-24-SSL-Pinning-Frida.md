# Day 24: The Jailbreak (SSL Pinning)

**Objective:** Force an App to trust your MITM Certificate.
**Tool:** Frida, Objection.

---

## 1. The Problem
Android Browser trusts User Certificates (Day 23).
**Apps DO NOT.**
Most apps have a list of valid certificates hardcoded inside the APK.
*   They connect.
*   They see your MITM Cert.
*   They check the list: "MITM Cert != Valid Cert".
*   They cut the connection.

**Result:** You see "Connection Error" in the app, and NO traffic in MITMProxy.

## 2. The Solution (Frida)
We cannot change the hardcoded list easily.
But we can change the **Running Code** in RAM.
**Frida** acts like a debugger. It pauses the app, finds the "Check Certificate" function, and rewrites it to `return true` (Valid).

## 3. Preparation
1.  **Install Frida Tools (PC):**
    ```bash
    pip install frida-tools objection
    ```
2.  **Install Frida Server (Phone):**
    *   Download `frida-server` (android-x8/x64 depending on emulator).
    *   Push to phone: `adb push frida-server /data/local/tmp/`.
    *   Run it: `adb shell`, `chmod +x ...`, `./frida-server &`.

## 4. The Magic Command (Objection)
Objection is a wrapper that contains pre-written scripts for 99% of apps.

1.  Start the app (e.g., `com.tinder`).
2.  Connect Objection:
    ```bash
    objection -g com.tinder explore
    ```
3.  Disable Pinning:
    ```bash
    android sslpinning disable
    ```
4.  **Watch the magic.**
    The CLI will say "Job: Found Cloudflare... Found OkHttp... Hooked!".
    The app should now load data.
    MITMProxy should light up with JSON.

## 5. Homework
1.  Download a secured app (Twitter, Instagram, or a Banking App - just for test).
2.  Try to intercept it. (It will fail).
3.  Use Objection.
4.  See if it passes. (Note: Financial apps have extra protections, don't worry if they fail).
