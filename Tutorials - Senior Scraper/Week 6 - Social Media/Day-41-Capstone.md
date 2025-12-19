# Day 41: The Social Graph (Capstone)

**Objective:** Build a robust Profile Scraper that doesn't die after 5 requests.
**Strategy:** Avoid "Login" at all costs. Use Caches.

---

## 1. The "Google Cache" Bypass
LinkedIn blocks frequent profile views. Google does not.
If you scan `http://webcache.googleusercontent.com/search?q=cache:linkedin.com/in/USERNAME`, you get the full HTML of the profile as Google saw it yesterday.

*   **Pros:** 100% Ban Safe. No Login required.
*   **Cons:** Data might be 3-5 days old.

## 2. The Logic
```python
from curl_cffi import requests
from bs4 import BeautifulSoup
import re

def get_profile_safe(username):
    # Strategy 1: Attempt Direct Public Profile (Mobile User Agent)
    # Strategy 2: Fallback to Google Cache
    
    url = f"https://linkedin.com/in/{username}"
    print(f"Trying Direct: {url}")
    
    headers = {
        "User-Agent": "Mozilla/5.0 (Linux; Android 10; K) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Mobile Safari/537.36",
        "Accept-Language": "en-US,en;q=0.9"
    }
    
    try:
        r = requests.get(url, headers=headers, impersonate="chrome124", timeout=10)
        
        if "authwall" in r.url:
            print("⚠️ Authwall hit. Switching to Cache Strategy...")
            return get_profile_cache(username)
            
        return parse_linkedin(r.text)
        
    except Exception as e:
        print(f"Error: {e}")
        return None

def get_profile_cache(username):
    print(f"Trying Cache for {username}...")
    # Note: Google Cache URL format changes occasionally
    url = f"http://webcache.googleusercontent.com/search?q=cache:https://www.linkedin.com/in/{username}"
    
    # Google Cache requires a Desktop UA usually
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)..."}
    
    r = requests.get(url, headers=headers)
    if "404" in r.text:
        print("❌ Not in Google Cache.")
        return None
        
    return parse_linkedin(r.text)

def parse_linkedin(html):
    soup = BeautifulSoup(html, "html.parser")
    
    # Easy extraction: Look for the <title> tag
    # "Elon Musk - CEO - Tesla | LinkedIn"
    title_tag = soup.find("title").string if soup.find("title") else ""
    
    # JSON-LD Extraction (The Pro Move)
    # See Day 7 / Day 36
    
    return {"raw_title": title_tag}

if __name__ == "__main__":
    # Test with a famous profile
    data = get_profile_safe("satyanadella")
    print(data)
```

## 3. Account Rotation (If you MUST login)
If you need email addresses (not in public profile), you must login.
**The Architecture:**
1.  **Redis Queue:** `jobs:scrape_profile`.
2.  **Worker Pool:** 10 Python scripts running.
3.  **Account Bank:** Each worker picks an account from `accounts.json`.
4.  **Cooldown:** Limit each account to 5 profiles per hour. Use `redis.set(account_id, 'cooldown', ex=3600)`.

## 4. Graduation
You have survived **Week 6 (Social Media)**.
You know the "Big 3" blocking risks:
1.  **IP Blocks** (Clouds - Week 1).
2.  **TLS Fingerprints** (Datadome - Week 2).
3.  **Account Bans** (LinkedIn - Week 6).

**Next Up (Week 7):** Building the API (Productizing your scripts).
