# Day 2: The Evasion Implementation

**Objective:** Bypass a live Cloudflare "I'm under attack" page using `curl_cffi`.
**Target:** We will use `https://gitlab.com/users/sign_in` (often has bot protection) or similar Cloudflare test sites.

---

## 1. The "StealthRequest" Class

Don't write raw code. Let's build a reusable class you can use in future projects.

Create a file named `stealth_scraper.py`:

```python
from curl_cffi import requests
from typing import Dict, Optional

class StealthScraper:
    def __init__(self, browser_type: str = "chrome124"):
        self.browser_type = browser_type
        self.session = requests.Session()
    
    def get(self, url: str, headers: Optional[Dict] = None):
        """
        Sends a GET request that looks exactly like a real browser.
        """
        # Default Headers that real Chrome sends
        default_headers = {
            "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8",
            "accept-language": "en-US,en;q=0.9",
            "cache-control": "max-age=0",
            "upgrade-insecure-requests": "1",
            "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
        }
        
        if headers:
            default_headers.update(headers)

        print(f"🕵️  Scraping {url} as {self.browser_type}...")
        
        response = self.session.get(
            url,
            headers=default_headers,
            impersonate=self.browser_type,
            timeout=30
        )
        
        return response

if __name__ == "__main__":
    # Test against a known heavyCloudflare site
    scraper = StealthScraper()
    
    # Try G2.com (Very hard target)
    target = "https://www.g2.com/products/slack/reviews"
    
    try:
        resp = scraper.get(target)
        if resp.status_code in [200, 404]:
            print("✅ SUCCESS! Accessed the page.")
            print(f"Title length: {len(resp.text)}")
        elif resp.status_code == 403:
            print("❌ FAILED. Blocked by Cloudflare.")
        else:
            print(f"⚠️ Status: {resp.status_code}")
    except Exception as e:
        print(f"Error: {e}")
```

## 2. Why "Impersonate" Works
The `impersonate="chrome124"` argument does all the heavy lifting.
It tells the underlying C library (curl) to:
1.  Use the exact Cipher Suite that Chrome 124 uses.
2.  Send the extensions in the exact order.
3.  Use HTTP/2 instructions identical to Chrome.

## 3. Handling "Just-In-Time" Challenges
Sometimes, even with this, you get a 403. Why?
**Cookies.**
Cloudflare often runs a JavaScript challenge *before* letting you in. Since `curl_cffi` is NOT a browser (it doesn't execute JS), it fails if the site requires a JS calculation.

**The Hybrid Approach (The "Warm Up"):**
1.  If the request fails with `curl_cffi`...
2.  Spin up a **Headless Browser** (Day 3 topic) just once.
3.  Solve the challenge.
4.  Extract the `cf_clearance` cookie.
5.  Inject that cookie into your `StealthScraper`.
6.  Close the browser and continue with fast Python requests.

## 4. Homework
1.  Run the script above against `dextools.io` or `opensea.io`.
2.  Try changing `browser_type` to `safari_15_3` and see if the result changes.
