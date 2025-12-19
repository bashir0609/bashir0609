# Day 29: The Cartographer (Google Maps)

**Objective:** Scrape business details from Google Maps.
**Difficulty:** Extreme.
**Why?**
1.  **Dynamic Classes:** Classes look like `class="A73 uD"`. They change every week.
2.  **Virtual Scroll:** You can't just "scroll to bottom". The DOM only keeps ~20 items. If you scroll down, top items disappear.
3.  **Canvas/SVG:** Some data is rendered graphically.

---

## 1. The "ARIAs" Strategy (Beating Dynamic Classes)
Never use classes like `.box-123` on Google.
Google (for legal/accessibility reasons) MUST keep **ARIA labels** (`aria-label`, `role`, `alt`) consistent for screen readers.

*   **Bad:** `div.m6QErb` (Will break next week).
*   **Good:** `div[role="feed"]` (The sidebar scroll container).
*   **Good:** `a[aria-label^="Visit"]` (The website link).

## 2. The Virtual Scroll Handling
Google Maps only renders the ~20 results you can "see" in the sidebar.
To get 100 results, you must:
1.  Hover the sidebar.
2.  Scroll specifically the sidebar element (not the window).
3.  Store scraped IDs to avoid duplicates (since elements are recycled).

## 3. The Code (Playwright)

```python
from playwright.sync_api import sync_playwright
import time

def scrape_maps(search_query):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=False)
        page = browser.new_page()
        
        # 1. Go to Maps
        page.goto("https://www.google.com/maps")
        
        # 2. Type Search
        page.locator("input#searchboxinput").fill(search_query)
        page.keyboard.press("Enter")
        time.sleep(3) # Wait for load (Use wait_for_selector in prod)
        
        # 3. Find the scrollable Feed
        # Note: This role="feed" is highly stable.
        feed = page.locator('div[role="feed"]')
        
        # 4. Scroll Loop
        results = set()
        
        for i in range(5):
            print(f"🔄 Scrolling... batch {i}")
            
            # Send 'PageDown' key to the Feed container
            feed.hover()
            page.mouse.wheel(0, 2000)
            time.sleep(2)
            
            # 5. Extract Visible Items
            # We look for links that take you to a place detail
            # Usually strict structure is: a > div > div.fontHeadlineSmall (Name)
            
            cards = page.locator("a.hfpxzc").all() # This class is relatively common for the invisible overlay link
            # BUT BETTER: Use the Aria Label on the link
            
            for card in cards:
                name = card.get_attribute("aria-label")
                link = card.get_attribute("href")
                if name and link:
                    results.add(f"{name} | {link}")
        
        print(f"✅ Found {len(results)} Unique Businesses:")
        for r in results:
            print(r)
            
        browser.close()

if __name__ == "__main__":
    scrape_maps("Plumbers in New York")
```

## 4. Why this script is simplistic
In a "Senior" script (Day 34 Capstone), you don't just grabbing the name.
You click the item, wait for the **Details Panel** to load, then extract:
*   Phone Number (`button[data-tooltip="Copy phone number"]`) -> Aria label again!
*   Website.
*   Reviews.

## 5. Homework
1.  Run the script.
2.  Modify it to extract the **Phone Number**.
    *   *Hint:* You can't see phone numbers on the list. You must Click the result -> Find the phone icon in the right panel -> Get text.
