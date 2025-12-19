# Day 3: Browser Perfection (Playwright)

**Objective:** Use a real browser (Playwright) without looking like a robot.
**Tools:** `playwright`, `playwright-stealth`.

---

## 1. The "Webdriver" Flag
By default, when you launch Chrome via code (Selenium/Playwright), it sets a javascript variable:
`navigator.webdriver = true`

Any anti-bot script checks this 1ms after the page loads. If true = BLOCK.

## 2. Setting up Stealth Playwright

We need to patch the browser *before* it travels to the URL.

```python
from playwright.sync_api import sync_playwright
import time
import random

def stealth_browser():
    with sync_playwright() as p:
        # 1. Launch args to hide automation features
        args = [
            '--disable-blink-features=AutomationControlled',
            '--start-maximized',
            '--disable-infobars',
        ]
        
        browser = p.chromium.launch(
            headless=False, # Use False for debugging, True for production
            args=args
        )
        
        # 2. Context level evasion
        context = browser.new_context(
            viewport={'width': 1920, 'height': 1080},
            user_agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36',
            locale='en-US'
        )
        
        # 3. The "Init Script" - This runs BEFORE page load
        # This manually overwrites the navigator.webdriver flags
        context.add_init_script("""
            Object.defineProperty(navigator, 'webdriver', {
                get: () => undefined
            });
            window.navigator.chrome = {
                runtime: {},
            };
            Object.defineProperty(navigator, 'languages', {
                get: () => ['en-US', 'en']
            });
            Object.defineProperty(navigator, 'plugins', {
                get: () => [1, 2, 3, 4, 5]
            });
        """)

        page = context.new_page()
        
        print("🕵️  Navigating to detection test...")
        # Test 1: SannySoft (A famous bot detector)
        page.goto("https://bot.sannysoft.com/")
        
        time.sleep(5)
        page.screenshot(path="stealth_test_result.png")
        print("📸 Screenshot saved. Check if everything is green.")

        browser.close()

if __name__ == "__main__":
    stealth_browser()
```

## 3. Human-Like Behavior
Even if you patch the flags, if you click immediately, you get caught.
**Bezier Curves:** Don't move the mouse in a straight line.
(We will cover this in Day 5).

## 4. Homework
1.  Run the code above.
2.  Open `stealth_test_result.png`.
3.  Look for "WebDriver" -> **False** (Green).
4.  Look for "Chrome" -> **Present** (Green).
5.  If you see "Intl" red errors, add the `locale` and `timezone_id` to `new_context()`.
