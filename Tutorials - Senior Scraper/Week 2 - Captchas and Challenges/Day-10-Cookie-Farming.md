# Day 10: "The Pass" (Cookie Farming)

**Objective:** Get verified ONCE in a browser, then use that "Hall Pass" (Cookie) in your fast Python script for 30 minutes.
**Why?** Solving Captchas in every request is slow and expensive. Solving it once and reusing the session is smart.

---

## 1. The Strategy
1.  **Browser:** Open Playwright (Non-Headless).
2.  **Pass:** Navigate to site. Solve cloudflare challenge / Captcha manually or via plugin.
3.  **Export:** Save the cookies to a JSON file.
4.  **Script:** Load JSON cookies into `requests` / `curl_cffi`.
5.  **Profit:** Make 1000 fast requests before the cookie expires.

## 2. Step 1: Farm and Export (Playwright)

```python
from playwright.sync_api import sync_playwright
import json

def farm_cookies(url):
    with sync_playwright() as p:
        # Use a persistent context so we keep cache/localstorage
        browser = p.chromium.launch_persistent_context(
            user_data_dir="./chrome_profile",
            headless=False
        )
        
        page = browser.new_page()
        page.goto(url)
        
        print("🛑 Please solve the captcha manually if it appears...")
        input("Press Enter when you are LOGGED IN or through the detailed challenge...")
        
        # Get Cookies
        cookies = browser.cookies()
        
        # Filter for the important ones (optional)
        # For Cloudflare, we need 'cf_clearance' and user agent
        
        with open("cookies.json", "w") as f:
            json.dump(cookies, f, indent=2)
            
        print("✅ Cookies saved to cookies.json")
        browser.close()

if __name__ == "__main__":
    farm_cookies("https://nowsecure.nl") # A test site
```

## 3. Step 2: Use the Cookies (Requests)

```python
import json
import requests

def use_cookies(url):
    # Load cookies
    with open("cookies.json", "r") as f:
        cookie_list = json.load(f)
    
    # Format for requests
    session = requests.Session()
    for cookie in cookie_list:
        session.cookies.set(cookie['name'], cookie['value'], domain=cookie['domain'])
    
    # CRITICAL: You must use the SAME User-Agent as the browser that farmed them.
    # Ideally save User-Agent in the json too. 
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) ..." 
    }
    
    resp = session.get(url, headers=headers)
    print(f"Status: {resp.status_code}") # Should be 200, not 403

```

## 4. The "Refresh" Loop
Advanced Scrapers automate this:
1.  Worker A (Browser) runs every 30 mins to get a new cookie. Writes to Redis.
2.  Worker B (Scraper) reads Cookie from Redis. If 403, alert Worker A to refresh.

## 5. Homework
1.  Run the farming script on a site like `g2.com`.
2.  Start a new script, load the `cf_clearance` cookie.
3.  Request a page. If you get 200, you have successfully "Farmed".
