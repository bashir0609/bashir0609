# Day 8: The "Kick Me" Sign (Webdriver Flags)

**Objective:** Understand why standard Selenium gets banned instantly.
**Concept:** `navigator.webdriver` & CDC variables.

---

## 1. The Simplest Check
Open your browser console (F12) right now and type:
```javascript
navigator.webdriver
```
*   **Real Chrome:** Returns `false` (or undefined).
*   **Selenium/Puppeteer:** Returns `true`.

Anti-bot scripts check this **1 millisecond** after page load. If it's true, you are flagged as a bot.

## 2. The `cdc_` String (The "Backdoor")
Even if you set `navigator.webdriver = false` using JavaScript injection, you can still be detected.
Standard `chromedriver.exe` contains a hardcoded variable named `cdc_adoQpoasnfa76pfcZLmcfl_`.
*   This variable is injected into *every* frame the browser opens.
*   Anti-bots scan the global window object for any variable starting with `cdc_`.
*   If found -> **BLOCK**.

## 3. How to fix it?

### A. The Patch (Hard Way)
You can open `chromedriver.exe` in a Hex Editor (like HxD) and replace `cdc_...` with `dog_...` (must be same length).
This works, but it's annoying to do for every update.

### B. The Flag (Easy Way)
Modern automation tools allow you to disable the automation extension.
```python
options.add_argument("--disable-blink-features=AutomationControlled")
```
This hides `navigator.webdriver`.

## 4. Detection Test
1.  Go to [sannysoft.com/test](https://bot.sannysoft.com/).
2.  Run standard Selenium.
3.  You will see **WebDriver: present (red)**.
4.  You will see **Connection: Headless (red)**.

## 5. Homework
1.  Write a script using standard Selenium.
2.  Visit Sannysoft.
3.  Take a screenshot.
4.  Try to add the `disable-blink-features` argument.
5.  Check if it turns green.
