# Day 21: The Heist (Capstone: Twitter Guest Token)

**Objective:** Reverse Engineer the famous "Guest Token" flow used by Twitter (and many others).
**Why?** This lets you scrape public tweets WITHOUT logging in.

---

## 1. The Recon
1.  Open Incognito Window. Go to `twitter.com/elonmusk`.
2.  Open DevTools > Network > Fetch/XHR.
3.  Refresh.
4.  Look for the request that gets the tweets (`UserByScreenName` or `UserTweets`).
5.  Look at the **Headers** of that request.
    *   `authorization`: `Bearer AAAAAAAAAAAAAAAAAAAAANRILgAAAAAAnNwIzUejRCOuH5E6I8xnZz4puTs%3D1Zv7ttFK8LfMDIA...` (This looks static/hardcoded).
    *   `x-guest-token`: `16584209384...` (This looks dynamic/numeric).

**Hypothesis:** The `authorization` header is public (hardcoded in the app), but the `x-guest-token` is generated fresh. We need to find HOW to get that numeric token.

## 2. Finding the Source
1.  Clear Network Log.
2.  Refresh.
3.  Look for the FIRST request that returns that numeric token.
4.  Search for `16584209` (the token value) in the "Response" content of previous requests.
5.  **Found it!** There is a POST request to `https://api.twitter.com/1.1/guest/activate.json`.
    *   Response: `{"guest_token":"16584209384..."}`

## 3. Replicating the "Activate" Request
Now check the request that *got* the token.
1.  It was a `POST` to `/activate.json`.
2.  Headers: It sent the `Authorization: Bearer AAAA...` string.
3.  Body: Empty?

**Conclusion:**
To get a guest token, you just need to `POST` to that endpoint with the hardcoded Bearer token.

## 4. The Python Solution (No Selenium)

```python
import requests
import json

# 1. The Hardcoded Bearer Token
# This key has been the same for 5+ years. It's in their main.js file.
BEARER_TOKEN = "Bearer AAAAAAAAAAAAAAAAAAAAANRILgAAAAAAnNwIzUejRCOuH5E6I8xnZz4puTs%3D1Zv7ttFK8LfMDIAIVwv6ZNWg3JUuSBznUzN4uhLlFN2xKtBN1"

def get_guest_token(session):
    headers = {
        "authorization": BEARER_TOKEN,
        "content-type": "application/x-www-form-urlencoded"
    }
    
    # 2. Call the Activate Endpoint
    print("🔑 Requesting Guest Token...")
    resp = session.post("https://api.twitter.com/1.1/guest/activate.json", headers=headers)
    
    if resp.status_code != 200:
        print("Failed to get token:", resp.text)
        return None
        
    token = resp.json().get("guest_token")
    print(f"✅ Got Token: {token}")
    return token

def scrape_user(username):
    session = requests.Session()
    
    # Step 1: Get the token
    guest_token = get_guest_token(session)
    if not guest_token: return

    # Step 2: Use the token to get Data
    # Note: Twitter GraphQL endpoints change often. This ID is an example.
    target_url = "https://twitter.com/i/api/graphql/UserByScreenName"
    
    headers = {
        "authorization": BEARER_TOKEN,
        "x-guest-token": guest_token,
        "content-type": "application/json"
    }
    
    variables = {"screen_name": username, "withSafetyModeUserFields": True}
    
    # Usually need query params here matching the variables
    # This part requires deeper inspection of specific GraphQL query IDs
    print(f"📡 scraping data for {username} using guest token...")
    # resp = session.get(url, headers=headers, params=...)
    # ...

if __name__ == "__main__":
    scrape_user("elonmusk")
```

## 5. Summary
You performed a "Chain":
1.  Found the Hardcoded Key (`Bearer`).
2.  Used Key -> To get Dynamic Session (`Guest Token`).
3.  Used Session -> To get Data (`UserTweets`).

This is the fundamental pattern of all API Scraping.

**Congratulations.** You have finished Week 3. You are now a text-based Reverse Engineer.
Next week: **Mobile Apps**.
