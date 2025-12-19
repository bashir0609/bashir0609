# Day 7: The "Hard Target" (G2.com Solution)

**Objective:** Scrape reviews from a G2 product page.
**Defense:** G2 uses heavy Cloudflare, Rate Limiting, and CSS obfuscation.
**Approach:** We will use `curl_cffi` (Day 2) + `BeautifulSoup`. No browser needed if our TLS is good.

---

## 1. The Strategy
1.  **Impersonate:** Use `chrome124`.
2.  **Headers:** Copy FULL headers from Chrome DevTools (Network Tab > Doc request > Copy as cURL).
3.  **Extraction:** The HTML is messy. We need robust selectors.

## 2. The Code

```python
from curl_cffi import requests
from bs4 import BeautifulSoup
import json
import time
import random

def scrape_g2_reviews(product_url: str):
    session = requests.Session()
    
    # Headers are CRITICAL here. G2 checks Referer and Origin.
    headers = {
        "authority": "www.g2.com",
        "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8",
        "accept-language": "en-US,en;q=0.9",
        "cache-control": "max-age=0",
        "referer": "https://www.google.com/",
        "upgrade-insecure-requests": "1",
        "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36",
    }

    print(f"🚀 Scraping: {product_url}")
    
    response = session.get(
        product_url,
        headers=headers,
        impersonate="chrome124",
        timeout=30
    )
    
    if response.status_code != 200:
        print(f"❌ Failed: {response.status_code}")
        # If 403: Your TLS fingerprint or Headers are wrong.
        return

    # Success? Let's parse.
    soup = BeautifulSoup(response.text, "html.parser")
    
    # Reviews are usually in a container like 'div[itemprop="review"]'
    # Note: G2 changes classes often. We look for Semantic HTML if possible.
    reviews = []
    
    # Try finding review cards
    review_cards = soup.find_all("div", class_="paper")  # "paper" is a common class on G2, might change
    
    if not review_cards:
        # Fallback: Look for schema.org data (JSON-LD)
        # This is the "Pro Move". Hidden JSON in the page often contains the data cleanly.
        print("⚠️ No review cards found via CSS. Checking JSON-LD...")
        scripts = soup.find_all("script", type="application/ld+json")
        for script in scripts:
            try:
                data = json.loads(script.string)
                if "@type" in data and "Review" in str(data):
                    print("✅ Found JSON-LD Review Data!")
                    # In a real app, parse this JSON.
                    # It creates a list of dicts with 'author', 'reviewRating', 'reviewBody'
                    print(json.dumps(data, indent=2)[:500] + "...")
                    return
            except:
                pass
        
        print("❌ Could not extract reviews. View source to update selectors.")
        return

    print(f"✅ Found {len(review_cards)} review elements (Visual).")
    
    for card in review_cards[:5]:
        # Extract name, text, rating...
        # This part requires manual inspection of the current live G2 site
        pass

if __name__ == "__main__":
    # Example: Slack reviews
    target = "https://www.g2.com/products/slack/reviews"
    scrape_g2_reviews(target)
```

## 3. Why JSON-LD?
In the code above, the CSS selector `div.paper` might break tomorrow.
However, **JSON-LD** (`application/ld+json`) is used for Google SEO. G2 WANTS Google to read this, so they rarely obfuscate it.
**Always look for JSON-LD first.** It's structured, clean, and hidden.

## 4. Week 1 Verification
If you can run this script and get a `200 OK` from G2.com, you have passed Week 1.
You have successfully spoofed TLS and bypassed one of the strictest Cloudflare settings on the web.

**Congratulations.** Next week, we stop using browsers entirely and verify Mobile APIs.
