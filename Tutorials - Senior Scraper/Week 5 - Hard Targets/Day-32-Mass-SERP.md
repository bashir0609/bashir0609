# Day 32: The Firehose (Mass Scaling SERP)

**Objective:** Scrape 10,000 keywords from Google Search.
**Difficulty:** High (Captchas).
**Tools:** `curl_cffi`, `proxies`.

---

## 1. Google's Defenses
Google doesn't just block IPs. They serve:
1.  **Consent Cookies:** "Before you continue..." (Europe).
2.  **Captchas:** "Traffic lights" (If you query too fast).
3.  **Soft Blocks:** Returning 0 results instead of error.

## 2. Magic Parameters
Do not scrape `google.com/search?q=pizza`.
Use the power parameters:
*   `num=100`: Get 100 results per request (Reduces requests by 10x!).
*   `gl=us`: Geo-Location (United States).
*   `hl=en`: Host Language (English).
*   `tbm=nws`: Search News (optional).

## 3. The "UULE" Cookie (Location Spoofing)
If you want to search "Pizza" as if you are in *Paris*, but your Proxy is in *New York*, Google will be confused.
**UULE** is a magic parameter that tells Google exactly where you are (GPS coordinates) encoded in Base64.
*   There are online "UULE Generators".
*   Add `&uule=w+CAIQICI...` to your URL to force local results.

## 4. The Code (Headless)
We use `curl_cffi` because Google fingerprinting is extremely strict on HTTP/2.

```python
from curl_cffi import requests
from bs4 import BeautifulSoup
import time
import random

def scrape_google(query, num=100):
    # Formatted URL
    query_clean = query.replace(" ", "+")
    url = f"https://www.google.com/search?q={query_clean}&num={num}&hl=en&gl=us"
    
    print(f"🔍 Searching: {query}")
    
    # Session with Chrome Impersonation
    session = requests.Session()
    
    headers = {
        "authority": "www.google.com",
        "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8",
        "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36",
    }
    
    # Ideally use a Rotating Proxy here
    # proxies = {"http": "...", "https": "..."}
    
    r = session.get(url, headers=headers, impersonate="chrome124")
    
    if "sorry/index" in r.url:
        print("❌ CAPTCHA DETECTED. Rotate Proxy.")
        return []
        
    # Parsing
    soup = BeautifulSoup(r.text, "html.parser")
    
    results = []
    
    # Review the DOM for 'g' class (The result container)
    for g in soup.find_all("div", class_="g"):
        # Anchor tag
        link = g.find("a")
        if link and link.get("href") and "http" in link.get("href"):
             # Title usually h3
             title = g.find("h3")
             if title:
                 results.append({
                     "title": title.text,
                     "url": link.get("href")
                 })
                 
    return results

if __name__ == "__main__":
    data = scrape_google("site:linkedin.com/in/ python developer", num=10)
    print(f"✅ Found {len(data)} results:")
    for d in data[:5]:
        print(d)
```

## 5. Homework
1.  Run the script.
2.  Search for `site:linkedin.com/in/ "software engineer" "open to work"`.
3.  This is called "X-Ray Search". You are now scraping LinkedIn profiles *without* logging into LinkedIn, by using Google as a proxy.
