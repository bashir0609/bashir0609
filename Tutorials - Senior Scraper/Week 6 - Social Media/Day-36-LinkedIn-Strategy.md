# Day 36: The Fortress (LinkedIn Account Safety)

**Objective:** Scrape LinkedIn without getting your account banned (or jailed).
**Difficulty:** Extreme (Security).
**Rule #1:** NEVER scrape with your personal LinkedIn account.

---

## 1. The Ban Hammer
LinkedIn tracks everything:
1.  **Page Views:** If you view 100 profiles in 10 minutes -> **Commercial Use Limit**.
2.  **Connections:** If you send 50 requests -> **Account Restriction**.
3.  **IP Address:** If you log in from a DC proxy -> **Instant Ban**.

## 2. The Strategy: "Public" vs "Private"

### Method A: The Private (Logged In) Way
*   **Pros:** You see everything (Email, Phone, Full history).
*   **Cons:** High risk. You need **Aged Accounts**.
*   **Cost:** You must buy accounts ($5-$10 each) and high-quality Residential Proxies.
*   **Limit:** ~50-80 profiles per day per account.

### Method B: The Public (Logged Out) Way
*   **Pros:** Zero risk to your account (you don't log in).
*   **Cons:** You see limited info. No email.
*   **How:** Google Cache, Bing Cache, or Mobile User-Agents.

## 3. Account Warming (If using Method A)
You cannot buy an account and start scraping. You must **Warm** it.
*   **Day 1:** Log in (Resi Proxy). Scroll feed. Like 1 post.
*   **Day 2:** Add 1 connection. View 2 profiles.
*   **Day 3:** View 5 profiles.
*   **Day 7:** You can start scraping (slowly).

## 4. Automation Code (Public Profile Strategy)
Instead of logging in, we pretend to be a Google Bot or generic mobile user.

```python
from curl_cffi import requests
from bs4 import BeautifulSoup

def scrape_public_profile(public_identifier):
    # url: https://www.linkedin.com/in/williamhgates
    url = f"https://www.linkedin.com/in/{public_identifier}"
    
    # LinkedIn allows search engines to see a "Preview" version
    headers = {
        "User-Agent": "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)",
        "Accept-Language": "en-US,en;q=0.9",
    }
    
    # Use DC or Resi proxy
    # proxies = {"https": "..."} 

    r = requests.get(url, headers=headers, impersonate="chrome124")
    
    if "authwall" in r.url or "login" in r.url:
        print("❌ Hit the Authwall. LinkedIn requires login for this IP.")
        return None
        
    soup = BeautifulSoup(r.text, "html.parser")
    
    # Parse the "JSON-LD" again! (See Day 7)
    # LinkedIn leaves structured data for Google to read.
    # Look for <script type="application/ld+json">
    return r.text[:500] # Debug

if __name__ == "__main__":
    print(scrape_public_profile("williamhgates"))
```

## 5. Homework
1.  Try the `Googlebot` user-agent on a profile.
2.  If it redirects to Login, your IP is flagged. Switch to a mobile 4G proxy.
3.  Or try using Google Cache: `http://webcache.googleusercontent.com/search?q=cache:linkedin.com/in/williamhgates`
