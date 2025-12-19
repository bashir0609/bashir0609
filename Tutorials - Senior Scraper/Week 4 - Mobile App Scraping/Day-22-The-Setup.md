# Day 22: The Lab Setup (MITMProxy + Android)

**Objective:** Create an environment where you can see the traffic of a Mobile App just like you see the traffic of a Website in Chrome DevTools.
**Tools:** MITMProxy, Android Emulator (Genymotion or LDPlayer).

---

## 1. The Concept: Man-In-The-Middle (MITM)
Normally: `Phone -> Server`
We want: `Phone -> YOUR PC (MITM) -> Server`

Your PC will act as a "Proxy". It will decrypt the HTTPS traffic, show it to you, and then send it to the server.

## 2. Install MITMProxy
1.  **Download:** Go to [mitmproxy.org](https://mitmproxy.org/) and download the Windows installer.
2.  **Run:** Open a terminal and type:
    ```bash
    mitmweb
    ```
3.  **Verify:** It should open a web interface in your browser (usually `http://127.0.0.1:8081`). This is your "Network Tab" for mobile.
4.  **Note the IP:** Open cmd and type `ipconfig`. Note your Local IPv4 (e.g., `192.168.1.50`).

## 3. Install Android Emulator
Do NOT use the default Android Studio AVD (It's slow and hard to root).
**Use Genymotion (Free for Personal Use) or LDPlayer.**

1.  **Genymotion:** Download and install.
2.  **Create Device:** Select a phone (e.g., Google Pixel 3) and an Android version (Android 10 is usually best/easiest to root).
3.  **Start** the phone.

## 4. Connect Phone to Proxy
Now tell the phone to send traffic to your PC.

1.  On the Android Emulator, go to **Settings** > **Wi-Fi**.
2.  Long press the current network ("WiredSSID" or similar).
3.  **Modify Network**.
4.  **Proxy:** Manual.
5.  **Proxy Hostname:** Your PC's IP (`192.168.1.50`).
6.  **Proxy Port:** `8080` (Default mitmproxy port).
7.  **Save.**

## 5. Install the Certificate (CRITICAL)
If you open Chrome on the phone now, you get "Connection not private". Why? Because MITMProxy is signing the traffic, and the phone doesn't trust MITMProxy.

1.  Open **Chrome** inside the Android Emulator.
2.  Go to `mitm.it`.
3.  You should see "Click here to download certificate".
4.  Click **Android**.
5.  Name it "mitm" and install it. (You might need to set a Lock Screen PIN).

## 6. Test
1.  Open any website in Android Chrome.
2.  Look at your PC (`mitmweb` interface).
3.  You should see the requests appearing!

**Homework:** Get this setup working. If you can see traffic from the Android Browser in your MITMWeb interface, you are ready for Day 24.
