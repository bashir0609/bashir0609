# Day 18: The Architect (Reverse Engineering Capstone)

**Objective:** Build a scraper that "heals" itself.
**Target:** A generic "Guest Token" flow (like Twitter/X).

---

## 1. The Pattern
Many APIs work like this:
1.  **Handshake:** Call `POST /activate` -> Get `token`.
2.  **Data:** Call `GET /feed` with `X-Guest-Token: token`.
3.  **expiry:** After 10 mins, `GET /feed` returns `403 Forbidden`.

A bad scraper crashes.
A Data Acquisition Engineer's scraper **auto-refreshes**.

## 2. The Implementation

```python
import requests
import time

class RobustScraper:
    def __init__(self):
        self.session = requests.Session()
        self.guest_token = None
        
        # Hardcoded secrets (Found via Day 17 AST)
        self.bearer_token = "Bearer AAAA..." 
    
    def refresh_token(self):
        print("🔄 Refreshing Guest Token...")
        # (Found via Day 15 Network Analysis)
        headers = {"Authorization": self.bearer_token}
        resp = self.session.post("https://api.example.com/activate", headers=headers)
        
        if resp.status_code == 200:
            self.guest_token = resp.json()['guest_token']
            print(f"✅ New Token: {self.guest_token}")
        else:
            raise Exception("Failed to get token")

    def get_data(self, user_id):
        # Retry Loop
        for attempt in range(2):
            if not self.guest_token:
                self.refresh_token()
                
            headers = {
                "Authorization": self.bearer_token,
                "X-Guest-Token": self.guest_token
            }
            
            resp = self.session.get(f"https://api.example.com/users/{user_id}", headers=headers)
            
            if resp.status_code == 200:
                print("✅ Data Received")
                return resp.json()
                
            elif resp.status_code == 403:
                print("⚠️ 403 Forbidden. Token expired?")
                # Clear token so next loop triggers refresh
                self.guest_token = None
                continue
            
            else:
                print(f"❌ Error: {resp.status_code}")
                return None
                
        return None

if __name__ == "__main__":
    bot = RobustScraper()
    # First call will trigger refresh
    data = bot.get_data("12345")
    
    # Simulate expiry if we waited 15 mins...
    # bot.get_data("67890")
```

## 3. Why this matters
You are not writing scripts. You are writing **Systems**.
*   **Separation of Concerns:** Token logic is separate from Scraping logic.
*   **Resilience:** It handles 403s gracefully.
*   **State:** It maintains the session.

## 4. Graduation
You have completed Week 3.
*   **Week 1:** Handshakes (TLS).
*   **Week 2:** Browsers (Patching).
*   **Week 3:** APIs (Reverse Engineering).

**Next Week:** We leave the web. We go to **Mobile Apps (MITM)**.
