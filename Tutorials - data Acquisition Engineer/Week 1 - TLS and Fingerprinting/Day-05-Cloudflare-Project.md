# Day 5: The Test (Cloudflare Project)

**Objective:** Scrape a site that is actively protected by Cloudflare.
**Target:** `https://www.g2.com/products/slack/reviews` OR `https://nowsecure.nl`.

---

## 1. The Strategy
We are not using Selenium. We are not using Playwright.
We are using `curl_cffi` to mimic Chrome 124 entirely in code.

**Checklist:**
1.  **Impersonate:** `chrome124`.
2.  **Headers:** Copy heavily from Real Chrome (Accept, Accept-Language, Sec-CH-UA).
3.  **HTTP/2:** Enabled by default in `curl_cffi`.

## 2. The Code

```python
from curl_cffi import requests
from bs4 import BeautifulSoup
import time

def scrape_protected_page(url):
    print(f"🕵️ Attempting to scrape: {url}")
    
    # 1. Real Chrome Headers (CRITICAL)
    # If you miss 'upgrade-insecure-requests' or 'sec-fetch-*', you might fail.
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36",
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
        "Accept-Language": "en-US,en;q=0.9",
        "Accept-Encoding": "gzip, deflate, br, zstd",
        "Upgrade-Insecure-Requests": "1",
        "Sec-Fetch-Dest": "document",
        "Sec-Fetch-Mode": "navigate",
        "Sec-Fetch-Site": "none",
        "Sec-Fetch-User": "?1",
        "Cache-Control": "max-age=0",
    }
    
    # 2. The Impersonation Request
    try:
        response = requests.get(
            url,
            impersonate="chrome124",
            headers=headers,
            timeout=30
        )
    except Exception as e:
        print(f"❌ Connection Error: {e}")
        return

    # 3. Validation
    if response.status_code == 403:
        print("❌ 403 Forbidden. Cloudflare blocked you.")
        print("Tip: Check your IP quality or header order.")
        
    elif response.status_code == 200:
        # Check if we got the "Just a moment..." page
        if "Just a moment" in response.text or "Enable JavaScript" in response.text:
            print("⚠️ 200 OK, but it's the Challenge Page.")
            # This happens if your JA3 is good, but your Behavior/Cookies are bad.
            # Day 10 (Cookie Farming) fixes this.
        else:
            print("✅ SUCCESS! Real content received.")
            print(f"Response Length: {len(response.text)} bytes")
            
            # Extract content to prove it
            soup = BeautifulSoup(response.text, "html.parser")
            print("Title:", soup.title.string.strip() if soup.title else "No Title")

if __name__ == "__main__":
    # Test 1: Just a detection test
    scrape_protected_page("https://nowsecure.nl") 
    
    print("-" * 20)
    
    # Test 2: G2 (Harder)
    scrape_protected_page("https://www.g2.com/products/slack/reviews")
```

## 3. Analysis of Failure
If it fails:
1.  **IP Reputation:** Are you using a Data Center IP? Cloudflare blocks those by default on strict sites.
2.  **Cipher Order:** Did you modify the `impersonate` string?
3.  **Header Integrity:** Did you remove a header like `Sec-Fetch-Dest`?

## 4. Graduation
You have completed Week 1 of the **Data Acquisition Engineer** path.
You understand the "Wire Layer" (TLS, HTTP/2, Packets).
Most scrapers only know the "HTML Layer". You are already deeper.

**Next Week:** Brute Force browser patching for when `curl_cffi` isn't enough.
