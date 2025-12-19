# Day 22: The Interceptor (MITMProxy Installation)

**Objective:** Inspect HTTP/HTTPS traffic flowing through your computer.
**Tool:** MITMProxy.

---

## 1. What is MITMProxy?
It is a "Man-In-The-Middle" proxy.
*   **Normal:** App -> Server.
*   **MITM:** App -> **Your PC** -> Server.

Because it sits in the middle, it can decrypt SSL traffic (if you install its certificate) and show you the raw JSON.

## 2. Installation (Windows)
1.  Go to [mitmproxy.org](https://mitmproxy.org/).
2.  Download the **Windows Installer**.
3.  Run it.

## 3. Running it
You use the command line, but it opens a Web Interface.

1.  Open PowerShell.
2.  Run: `mitmweb`
    *   *Note:* `mitmproxy` is the text-UI (hard to use on Windows). `mitmweb` is the Browser-UI.
3.  It should open `http://127.0.0.1:8081` in Chrome.
4.  This is your "Control Center".

## 4. Testing it
1.  Configure Windows or Firefox to use localhost:8080 as a proxy.
    *   *Easier Way:* Run Chrome with a proxy flag:
        ```bash
        "C:\Program Files\Google\Chrome\Application\chrome.exe" --proxy-server="127.0.0.1:8080"
        ```
2.  Go to `http://mitm.it`.
3.  If you see "Click to install certificate", IT WORKS.
4.  If you see "Site can't be reached", the proxy is not running.

## 5. Homework
1.  Get `mitmweb` running.
2.  Visit `example.com` through the proxy.
3.  See the request appear in the Web Interface.
