# Day 5: The "Click" Test (Human Mouse Movement)

**Objective:** Move the mouse like a human (curved, variable speed) to bypass behavioral analysis.
**Why?** Anti-bots track `mousemove` events. If your mouse jumps from (0,0) to (500,500) in 1ms, you are a bot.

---

## 1. The Theory: Bezier Curves
Robots move in lines ($y = mx + b$).
Humans move in arcs. Our wrist creates a pivot point, creating a natural curve.
We also accelerate and decelerate (Fitts's Law).

## 2. Using `pyautogui` logic or custom implementation
We generally inject JavaScript to visualize this, but in Playwright, we simulate it via the Protocol.

However, writing Bezier math from scratch is hard. Let's create a Helper Class involving **Non-Linear Ease**.

```python
import random
import time
import math
from playwright.sync_api import Page

def human_type(page: Page, selector: str, text: str):
    """Types text with random delays between keystrokes"""
    page.focus(selector)
    for char in text:
        page.keyboard.type(char)
        # Random delay between 50ms and 150ms
        time.sleep(random.uniform(0.05, 0.15))

def human_click(page: Page, selector: str):
    """
    Moves mouse to element with 'overshoot' and 'jitter'
    Then clicks with a small random delay.
    """
    box = page.locator(selector).bounding_box()
    if not box:
        print(f"Could not find {selector}")
        return

    # Calculate target center
    target_x = box['x'] + box['width'] / 2
    target_y = box['y'] + box['height'] / 2
    
    # Add randomness to the exact click point (humans rarely click dead center)
    target_x += random.uniform(-10, 10)
    target_y += random.uniform(-5, 5)

    # Move mouse
    # Playwright's mouse.move creates a straight line by default
    # To curve it, we would need to generate intermediate steps.
    # For now, let's use the 'steps' argument to slow it down.
    
    current_pos = page.mouse.position
    steps = 20 # More steps = slower, smoother
    
    page.mouse.move(target_x, target_y, steps=steps)
    
    # Pause before clicking (Human reaction time)
    time.sleep(random.uniform(0.1, 0.3))
    
    page.mouse.down()
    time.sleep(random.uniform(0.05, 0.1)) # Hold click briefly
    page.mouse.up()

# --- Advanced: The Ghost Cursor Library ---
# There is a famous library called "ghost-cursor" (JS port) for Python.
# It handles the Bezier math for you.
# Recommendation: For Day 5, focus on simple 'steps' + 'random delays'.
# If you are still blocked, search github for "playwright-ghost-cursor" python ports.
```

## 3. Practice Script

```python
from playwright.sync_api import sync_playwright

def test_humanity():
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=False)
        page = browser.new_page()
        
        # A site that tests clicking speed/accuracy
        page.goto("https://humanbenchmark.com/tests/reactiontime")
        
        print("Waiting for load...")
        time.sleep(2)
        
        # Use our human clicker
        print("Clicking...")
        human_click(page, ".view-splash") # The big green area
        
        print("Done. Did it feel robotic?")
        time.sleep(3)
        browser.close()

if __name__ == "__main__":
    test_humanity()
```

## 4. Homework
1.  Go to a "Login Page" (e.g. Gmail or Facebook).
2.  Use `human_type` to enter a fake email.
3.  Observe how it looks compared to `page.fill()`. `page.fill()` is instant (Robot). `human_type` is slow (Human).
