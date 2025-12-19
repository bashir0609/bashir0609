# Day 25: The Golden Endpoint (Simple Interception)

**Objective:** Extract data from a mobile app.
**Target:** A simple News or Weather app (Low security).

---

## 1. The Setup
1.  Emulator Running + Wifi Proxy set to PC.
2.  `mitmweb` running on PC.
3.  App Installed (e.g., "BBC News" or similar).

## 2. Filtering the Noise
Mobile apps are noisy. They send analytics to Google, Facebook, Crashlytics every 2 seconds.
In `mitmweb`, the list scrolls too fast.

**The Filter:**
Type this in the search bar:
`~t json & !~u google & !~u facebook`
*   `~t json`: Content-Type is JSON.
*   `!~u google`: URL does not contain "google".

## 3. The Extraction
1.  Open App.
2.  Refresh the "Feed".
3.  Look at MITMWeb.
4.  Find a GET request to `api.bbc.co.uk/...`.
5.  Click it -> Response Tab.
6.  Do you see the Headlines? **Found it.**

## 4. Coding the Scraper
1.  Click the Request in MITMWeb.
2.  Keep the **Headers**. Mobile APIs check `User-Agent` (e.g., `Dalvik/2.1.0...`) and `X-App-Version`.
3.  Write the Python script:

```python
import requests

url = "https://metrics.bbc.co.uk/..." # The URL you found
headers = {
    "User-Agent": "Dalvik/2.1.0 (Linux; U; Android 10; Pixel 3 Build/QQ3A.200805.001)",
    "X-BBC-Edge-Cache": "...",
}

resp = requests.get(url, headers=headers)
print(resp.json())
```

## 5. Homework
1.  Download a customized news app.
2.  Intercept the feed.
3.  Write a script to get the top 10 headlines.
