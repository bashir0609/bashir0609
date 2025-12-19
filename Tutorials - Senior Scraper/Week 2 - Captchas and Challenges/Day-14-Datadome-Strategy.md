# Day 14: The Final Boss (Datadome Strategy)

**Objective:** Combine everything (TLS, Headers, Proxies) to bypass **Datadome**.
**Example Targets:** Ticketmaster (Home page), Hermes, Footlocker.

---

## 1. The Datadome Checklist
Datadome is smart. If you fail ANY of these, you get blocked:
1.  **TLS Fingerprint:** Must match Chrome 100%. (Use `curl_cffi` or patched Playwright).
2.  **IP Address:** Must be Residential. (DC proxies are insta-banned).
3.  **Headers:** Must be perfect order.
4.  **Behavior:** If you request 10 pages in 1 second, you are banned.

## 2. All-in-One Implementation (Code)

```python
from curl_cffi import requests
import time
import random

# You NEED a residential proxy for this.
# Format: http://user:pass@host:port
RESIDENTIAL_PROXY = "http://username:password@us.smartproxy.com:10000"

def datadome_killer(url):
    print(f"⚔️ attack on {url}")
    
    # 1. The Session (TLS + Http2)
    session = requests.Session()
    
    # 2. Perfect Headers (Chrome 124)
    headers = {
        "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
        "accept-language": "en-US,en;q=0.9",
        "cache-control": "max-age=0",
        "sec-ch-ua": '"Chromium";v="124", "Google Chrome";v="124", "Not-A.Brand";v="99"',
        "sec-ch-ua-mobile": "?0",
        "sec-ch-ua-platform": '"Windows"',
        "sec-fetch-dest": "document",
        "sec-fetch-mode": "navigate",
        "sec-fetch-site": "none",
        "sec-fetch-user": "?1",
        "upgrade-insecure-requests": "1",
        "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
    }
    
    # 3. The Proxy
    proxies = {"http": RESIDENTIAL_PROXY, "https": RESIDENTIAL_PROXY}
    
    try:
        response = session.get(
            url,
            headers=headers,
            proxies=proxies,
            impersonate="chrome124",
            timeout=15
        )
        
        # 4. Check for Challenge
        if response.status_code == 403:
             if "datadome" in response.text:
                 print("☠️ Blocked by DataDome (Challenge Page).")
                 print("Advice: Your Proxy IP might be flagged, or TLS mismatch.")
             else:
                 print("❌ 403 Forbidden (Generic).")
                 
        elif response.status_code == 200:
             if "datadome" in response.text and "slider" in response.text:
                 print("⚠️ 200 OK, but got Captcha Slider.")
             else:
                 print("🎉 SUCCESS! We are in.")
                 print(response.text[:200])
                 
    except Exception as e:
        print(f"Error: {e}")

if __name__ == "__main__":
    datadome_killer("https://opentable.com") # OpenTable uses Datadome
```

## 3. What if I get the Slider?
If you get the Datadome Slider (Captcha):
1.  **Do NOT** try to solve it with `requests`. You won't have the JS context.
2.  Switch to **Playwright** (Day 3 Method).
3.  Use **Cookie Farming** (Day 10). Solve the slider ONCE in browser.
4.  Export the `datadome` cookie.
5.  Add that cookie to your Python script above.

## 4. Week 2 Verification
If you can scrape the homepage of a Datadome-protected site (like OpenTable or Hermes) and get the actual HTML (not the "Please Verify" page), you have graduated Week 2.

**Next Week:** We ditch the browser entirely and start "Hacking" mobile Ops.
