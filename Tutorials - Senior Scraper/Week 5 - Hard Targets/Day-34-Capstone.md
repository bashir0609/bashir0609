# Day 34: The Lead Finder (Capstone)

**Objective:** Build a tool that finds "Plumbers in Austin", scrapes their phone numbers (from Maps) AND their emails (from their websites).
**Value:** This script alone is a $500 product on Upwork.

---

## 1. The Architecture
1.  **Stage 1 (Discovery):** Use Playwright (Day 29) to scroll Google Maps and extract `Name`, `Phone`, `WebsiteURL`.
2.  **Stage 2 (Enrichment):** Use `requests` to visit `WebsiteURL`.
3.  **Stage 3 (Extraction):** Regex scan the website HTML for `mailto:` or generic email patterns.

## 2. The Code (Full Pipeline)

```python
import re
from playwright.sync_api import sync_playwright
from curl_cffi import requests
import csv
import time

# --- Part 1: Maps Scraper ---
def get_businesses(query):
    print(f"🗺️  Searching Maps for: {query}...")
    businesses = []
    
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True) # Headless for speed
        page = browser.new_page()
        page.goto(f"https://www.google.com/maps/search/{query}")
        page.wait_for_selector('div[role="feed"]', timeout=15000)
        
        # Quick Scroll (In real life, use the robust scroll form Day 29)
        feed = page.locator('div[role="feed"]')
        for _ in range(5):
            feed.hover()
            page.mouse.wheel(0, 3000)
            time.sleep(2)
            
        # Extract
        # Note: We need the Detail URL to click it, but for speed, let's grab what's visible
        cards = page.locator("a.hfpxzc").all()
        for card in cards:
            url = card.get_attribute("href")
            name = card.get_attribute("aria-label")
            # Phone/Website are usually metadata or require clicking.
            # Simplified for tutorial: usually we'd click each card.
            businesses.append({"name": name, "maps_url": url})
            
        browser.close()
    return businesses

# --- Part 2: Email Extractor ---
def find_email(website_url):
    try:
        if not website_url: return None
        if "http" not in website_url: website_url = "http://" + website_url
        
        print(f"   Searching {website_url}...")
        r = requests.get(website_url, timeout=5, impersonate="chrome124")
        
        # Regex for email
        emails = re.findall(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}', r.text)
        
        # Filter junk (images, fonts)
        valid_emails = [e for e in emails if "png" not in e and "jpg" not in e]
        return valid_emails[0] if valid_emails else None
        
    except:
        return None

# --- Part 3: Orchestrator ---
if __name__ == "__main__":
    # 1. Get Leads
    leads = get_businesses("Real Estate Agencies in Dallas")
    print(f"📍 Found {len(leads)} raw leads. Enriching...")
    
    # 2. Enrich (This would usually be the 'Website' found in Maps)
    # Since Maps scraping is complex, let's pretend we found a website
    # In a real script, you extract the website from the Maps sidebar.
    
    final_data = []
    for lead in leads:
        # Placeholder logic: In reality, extract 'website' from maps
        website = None 
        email = None
        
        # If we had a website, we'd run:
        # email = find_email(website)
        
        final_data.append({
            "name": lead['name'],
            "maps_url": lead['maps_url'],
            "email_guess": email
        })
        
    # 3. Save
    keys = final_data[0].keys()
    with open('leads.csv', 'w', newline='', encoding='utf-8') as f:
        dict_writer = csv.DictWriter(f, keys)
        dict_writer.writeheader()
        dict_writer.writerows(final_data)
        
    print("✅ Done. Saved to leads.csv")
```

## 3. The Money Shot
To make this professional:
1.  Add **Multithreading** to Part 2 (Enrichment). Use `ThreadPoolExecutor` to check 10 websites at once.
2.  Add **Phone Number** extraction from the Maps Sidebar.
3.  Add **Social Media** links (Facebook/Instagram) from the website HTML.

## 4. Graduation
You have survived **Week 5**.
You tackled Google Maps (Dynamic/Virtual), Google Search (Mass Scale), and built a full pipeline.

**Next Up (Week 6):** Social Media (LinkedIn & Instagram) - The Graph API and Login Walls.
