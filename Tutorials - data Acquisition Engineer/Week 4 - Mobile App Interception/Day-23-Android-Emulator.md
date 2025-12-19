# Day 23: The Victim (Android Emulator)

**Objective:** Set up a phone that we can "Spy" on.
**Tool:** Genymotion (Preferred) or LDPlayer.

---

## 1. Why Genymotion?
Standard Android Studio AVD is slow and hard to configure for Proxies.
Genymotion creates a VirtualBox VM that acts like a phone. It mimics a **real** wifi chip, making proxying easier.

## 2. Setup
1.  Download Genymotion (Free for Personal Use).
2.  Install VirtualBox (it comes with the installer usually).
3.  Create a new device:
    *   **Phone:** Google Pixel 3 (Standard size).
    *   **Android:** 10.0 (Perfect balance of compatibility and security).
4.  Start the device.

## 3. The Tunnel
Now we connect the Phone to MITMProxy.
1.  **Find your PC IP:** Open `cmd` -> `ipconfig`. Look for IPv4 (e.g., `192.168.1.50`).
2.  **On the Phone:**
    *   Settings -> Network & Internet -> Wifi.
    *   Your network is usually `AndroidWifi`.
    *   Long press (or Click Edit/Pencil icon).
    *   **Advanced Options**.
    *   **Proxy:** `Manual`.
    *   **Hostname:** `192.168.1.50` (YOUR PC IP).
    *   **Port:** `8080` (MITMProxy defaults).
    *   **Save.**

## 4. Trusting the Spy
If you open Chrome on the phone now, you get "Certificate Error".
1.  Open Chrome (on Phone).
2.  Go to `mitm.it`.
3.  Click "Android".
4.  Download the Certificate.
5.  Install it. (Name it: `mitm`).

## 5. Validation
1.  Open Chrome on Phone.
2.  Go to `google.com`.
3.  Check MITMWeb on your PC.
4.  Do you see the request? **Success.**

## 6. Homework
1.  Set up the emulator.
2.  Proxy the traffic.
3.  Install the Cert.
4.  Verify you can browse HTTPS sites without errors.
