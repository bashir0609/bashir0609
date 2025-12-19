# Day 28: The Mobile Heist (Capstone)

**Objective:** Build a scraper for an App that has **NO** website (or a very limited one).
**Example:** Tinder, Hinge, or specific "App-Only" deals on McDonald's.

---

## 1. The Workflow

**Step 1: The Environment** (Day 22)
*   Emulator: Genymotion Pixel 3 (Android 10).
*   Tool: MITMProxy running on port 8080.
*   Wifi: Configured to proxy to PC.

**Step 2: The Jailbreak** (Day 24)
*   App crashes on load? -> **SSL Pinning Detected.**
*   Action: Run `objection -g com.target.app explore`.
*   Command: `android sslpinning disable`.
*   Result: App loads data.

**Step 3: The Intercept** (Day 25)
*   Action: Perform a specific task (e.g., "Swipe Right" or "Load Deal").
*   Filter: `~t json`.
*   Found:`POST /api/v3/profile/like`.

**Step 4: The Analysis**
Look at the Body:
`{"user_id": "123456", "action": "like"}`

Look at the Headers:
`X-Auth-Token: 884402...`
`X-Sig: a1b2c3d4...` (Oh no, a signature?)

## 2. Handling App Signatures (Recap Week 3)
If the app uses a signature (`X-Sig`), you have two choices:
1.  **Decompile the APK:** Use `jadx-gui` to open the `.apk` file. Search for "X-Sig". Read the Java code. It's often easier than reading Minified Webpack JS!
2.  **Frida Hooking:** Use Frida to "Watch" the function that generates the signature and just ask the phone to sign it for you (Advanced).

## 3. The Python Code (The Deliverable)

```python
import requests
import uuid

class AppScraper:
    def __init__(self, auth_token):
        self.base_url = "https://api.targetapp.com/v1"
        self.headers = {
            "User-Agent": "TargetApp/4.2.0 (Android; 10; Pixel 3)",
            "X-Auth-Token": auth_token,
            "X-Device-ID": str(uuid.uuid4())
        }
    
    def get_feed(self):
        # We found this endpoint via MITMProxy Day 25
        endpoint = "/feed/nearby"
        
        r = requests.get(self.base_url + endpoint, headers=self.headers)
        
        if r.status_code == 200:
            return r.json()['items']
        else:
            print("Error:", r.text)
            return []

if __name__ == "__main__":
    # You got this token from MITMProxy headers
    TOKEN = "eyJh...12345" 
    
    app = AppScraper(TOKEN)
    data = app.get_feed()
    
    print(f"Found {len(data)} items straight from the API.")
```

## 4. Graduation
You now possess the skills to scrape:
1.  **Simple Sites** (Requests).
2.  **Protected Sites** (TLS Spoofing).
3.  **Complex Web Apps** (Hidden APIs).
4.  **Mobile Apps** (MITM + Frida).

**You are a Senior Scraper.**
The next 4 weeks (Weeks 5-8) are about Scale (Google/LinkedIn) and Monetization.
