# Day 8: Know Your Enemy (Captcha Types)

**Objective:** Identify which Captcha you are facing just by looking at the HTML.
**Why?** You can't solve it if you don't know what it is.

---

## 1. The Big Three

### A. Google reCAPTCHA (v2 and v3)
*   **Visual:** "I'm not a robot" checkbox (v2) or Invisible (v3).
*   **HTML Signature:** Look for an iframe with `src="google.com/recaptcha/..."` or a script loading `api.js`.
*   **Key Parameter:** `sitekey` (k=...). This is a public key you NEED to send to a solver.
*   **Behavior:**
    *   **v2:** You solve, you get a token, you allow the form.
    *   **v3:** You don't "solve" it. The browser sends a token, and Google returns a score (0.0 to 1.0) of how human you are. If you have bad IP/TLS, you get 0.1 and blocked.

### B. hCaptcha
*   **Visual:** Looks like reCaptcha but asks you to click on "Seaplanes" or "Pineapples". Used by Cloudflare.
*   **HTML Signature:** `hcaptcha.com/1/api.js` and a `div` with class `h-captcha`.
*   **Key Parameter:** `data-sitekey`.

### C. Geetest (The Slider)
*   **Visual:** A puzzle piece slider or clicking Chinese characters.
*   **HTML Signature:** Loads scripts from `geetest.com` or `gt.js`.
*   **Key Parameters:** `gt` and `challenge`.

## 2. Detection Script
Don't check manually. Write a helper to detect it.

```python
from bs4 import BeautifulSoup
import re

def detect_captcha(html_content):
    soup = BeautifulSoup(html_content, "html.parser")
    text = html_content.lower()
    
    # 1. reCAPTCHA
    if "google.com/recaptcha" in text or soup.find(class_="g-recaptcha"):
        sitekey = soup.find(class_="g-recaptcha")
        key = sitekey.get("data-sitekey") if sitekey else "Hidden in JS"
        return f"reCAPTCHA V2 detected! Sitekey: {key}"
        
    if "grecaptcha.execute" in text:
        return "reCAPTCHA V3 detected! (Invisible)"

    # 2. hCaptcha
    if "hcaptcha.com" in text or soup.find(class_="h-captcha"):
        sitekey = soup.find(class_="h-captcha")
        key = sitekey.get("data-sitekey") if sitekey else "Hidden in JS"
        return f"hCaptcha detected! Sitekey: {key}"

    # 3. Geetest
    if "geetest" in text or "gt.js" in text:
        return "Geetest detected! Look for 'gt' and 'challenge' values."
    
    # 4. Cloudflare Turnstile
    if "challenges.cloudflare.com" in text:
        return "Cloudflare Turnstile detected!"

    return "No obvious Captcha found."

# Test it
if __name__ == "__main__":
    fake_html = """
    <html>
        <body>
            <form action="/login">
                <div class="g-recaptcha" data-sitekey="6Lc_a-4SAAAAA..."></div>
            </form>
        </body>
    </html>
    """
    print(detect_captcha(fake_html))
```

## 3. Homework
1.  Go to a site like `2captcha.com/demo`.
2.  Inspect Element on the captcha.
3.  Find the `data-sitekey`. Copy it. You will need this for Day 9.
