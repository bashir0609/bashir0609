# Day 9: The Solver (2Captcha Implementation)

**Objective:** Programmatically solve the Captcha you found yesterday.
**Tools:** `requests`, `2captcha` (concept).

---

## 1. How Solvers Work
You do NOT click the images.
1.  You send the `sitekey` and the `page_url` to the 2Captcha API.
2.  Their human workers (or AI) solve it.
3.  They return a `token` (a long string of random characters).
4.  You insert this token into the hidden HTML input usually named `g-recaptcha-response`.
5.  You submit the form.

## 2. The Code

```python
import requests
import time

API_KEY = "YOUR_2CAPTCHA_KEY_HERE"

def solve_recaptcha(sitekey, url):
    print("🚀 Sending Captcha to 2Captcha...")
    
    # 1. Send Request
    resp = requests.post("http://2captcha.com/in.php", data={
        "key": API_KEY,
        "method": "userrecaptcha",
        "googlekey": sitekey,
        "pageurl": url,
        "json": 1
    })
    
    if resp.json().get("status") != 1:
        print("Error sending:", resp.text)
        return None
        
    request_id = resp.json().get("request")
    print(f"✅ Submitted. ID: {request_id}. Waiting for solution...")
    
    # 2. Poll for Solution
    for i in range(20):
        time.sleep(5) # Wait 5 seconds
        
        result_resp = requests.get(f"http://2captcha.com/res.php?key={API_KEY}&action=get&id={request_id}&json=1")
        result_json = result_resp.json()
        
        if result_json.get("status") == 1:
            token = result_json.get("request")
            print("🎉 SOLVED!")
            return token
            
        if result_json.get("request") == "CAPCHA_NOT_READY":
            print(".", end="", flush=True)
            continue
            
        print("Error:", result_json)
        break
        
    return None

def apply_token_playwright(page, token):
    # Depending on the site, there is a hidden textarea
    # We use JS to inject the token
    js_script = f"""
    document.getElementById('g-recaptcha-response').innerHTML = '{token}';
    """
    page.evaluate(js_script)
    print("💉 Token Injected. Submitting form...")
    
    # Sometimes you need to call a callback function too check the site code
    # page.evaluate("submitCallback()") # Example

if __name__ == "__main__":
    # Example usage
    my_token = solve_recaptcha("6Le-wvkSAAAAAPBMRTvw0Q4Muexq9bi0DJwx_mJ-", "https://google.com/recaptcha/api2/demo")
    print(f"Final Token: {my_token[:50]}...")
```

## 3. The "Gotcha" (DataDom / Enterprise Captchas)
2Captcha works great for Google and hCaptcha.
It works POORLY for **Datadome** or **Akamai** captchas because those require your TLS fingerprint to match the solver's fingerprint (which is impossible).

For those, you need **Cookie Farming** (Day 10).

## 4. Homework
1.  Register a demo account at 2Captcha/CapSolver.
2.  Use the script to solve the Google Demo page.
3.  Manually paste the token into the DevTools console to see if it verifies.
