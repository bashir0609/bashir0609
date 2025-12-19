# Day 12: The Ghost Driver (Capstone)

**Objective:** Build a Class that passes `pixelscan.net` (Green Checkmarks).
**Stack:** Selenium + Nodriver (Hybrid approach) or Pure Nodriver.

---

## 1. The Checklist
To pass a "fingerprint" scan, you need:
1.  **Network:** No leaking IPs (WebRTC).
2.  **Navigator:** No `webdriver` flag.
3.  **Canvas/Audio:** No unique/null footprints.
4.  **Consistency:** Your User-Agent OS must match your JavaScript `navigator.platform` OS.

## 2. The Code (Nodriver Implementation)
This is the modern way.

```python
import asyncio
import nodriver as uc
import random

async def stealth_browser():
    # 1. Start with randomized window size (avoids 800x600 detection)
    width = random.randint(1024, 1920)
    height = random.randint(768, 1080)
    
    browser = await uc.start(
        browser_args=[
            f"--window-size={width},{height}",
            "--disable-blink-features=AutomationControlled", # The Big One
            "--no-first-run",
            "--no-service-autorun",
            "--password-store=basic",
        ]
    )
    
    # 2. Page object
    page = await browser.get("https://pixelscan.net")
    
    # 3. Wait for analysis
    await asyncio.sleep(10)
    
    # 4. Check results
    content = await page.get_content()
    if "No automation framework detected" in content or "Consistent" in content:
        print("✅ PASSED Pixelscan")
    else:
        print("❌ FAILED Pixelscan")
        
    # Optional: Save screenshot
    await page.save_screenshot("pixelscan_result.png")
    
    return browser

if __name__ == "__main__":
    asyncio.run(stealth_browser())
```

## 3. The Code (Selenium + Stealth)
The "Classic" way for legacy support.

```python
from selenium import webdriver
from selenium_stealth import stealth

def create_stealth_driver():
    options = webdriver.ChromeOptions()
    options.add_argument("start-maximized")
    options.add_experimental_option("excludeSwitches", ["enable-automation"])
    options.add_experimental_option('useAutomationExtension', False)
    
    driver = webdriver.Chrome(options=options)
    
    stealth(driver,
        languages=["en-US", "en"],
        vendor="Google Inc.",
        platform="Win32",
        webgl_vendor="Intel Inc.",
        renderer="Intel Iris OpenGL Engine",
        fix_hairline=True,
    )
    
    return driver

if __name__ == "__main__":
    driver = create_stealth_driver()
    driver.get("https://bot.sannysoft.com/")
    input("Check browser...")
```

## 4. Graduation
You have completed Week 2.
*   **Week 1:** You learned to spoof the Network (TLS).
*   **Week 2:** You learned to spoof the Browser (Canvas/Audio/Drivers).

**Combined Power:**
You can now scrape:
1.  **API endpoints** (using TLS spoofing).
2.  **Complex JS sites** (using Patch Browser).

**Next Week:** We stop looking at the browser. We start looking at **Hidden APIs (Reverse Engineering)** to avoid using a browser entirely.
