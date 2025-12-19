# Day 9: The Ghost (Nodriver)

**Objective:** Use a browser automation tool that is detected as "Human" by default.
**Tool:** `nodriver` (Python).

---

## 1. Strategies: Patched vs Native
*   **Selenium-Stealth:** Tries to hide the `webdriver` flag using JS injection. It's a "Cat and Mouse" game.
*   **Undetected-Chromedriver:** Patches the binary (the hexadecimal edit from Day 8). Better, but still relies on Selenium.
*   **Nodriver:** Does NOT use Selenium/Webdriver at all. It uses the **Chrome DevTools Protocol (CDP)** directly.

**Why CDP wins:**
When you use CDP, you are basically "Remote Controlling" a real Chrome instance. There is no `chromedriver.exe` middleman injecting flags.
To the website, it looks exactly like a user verified via DevTools.

## 2. Installation
```bash
pip install nodriver
```

## 3. The Code
Nodriver is **Async-first**.

```python
import asyncio
import nodriver as uc

async def main():
    # Start the browser
    browser = await uc.start()
    
    # Go to a bot test
    page = await browser.get("https://bot.sannysoft.com")
    
    # Wait for check
    await asyncio.sleep(5)
    
    # Screenshot
    await page.save_screenshot("report.png")
    
    # Finding elements is different (it's text based often)
    # await page.find("Some Text").click()
    
    print("Done. Check report.png")
    # Unlike Selenium, you often don't "close" it immediately in testing
    # browser.stop()

if __name__ == "__main__":
    # Windows loop policy fix might be needed
    asyncio.run(main())
```

## 4. Pros & Cons
*   **Pros:** Passes Cloudflare/Datadome out of the box (mostly). Extremely fast.
*   **Cons:** Documentation is sparse. Syntax is different from Selenium (`find_element` vs `select`).

## 5. Homework
1.  Install `nodriver`.
2.  Run the script above against `nowsecure.nl`.
3.  Does it pass the Cloudflare Challenge? (It should).
