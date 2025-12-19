# Day 10: The Artists (Canvas Fingerprinting)

**Objective:** Prevent websites from identifying you by your Graphics Card.
**Concept:** Canvas Fingerprinting.

---

## 1. How it works
1.  Website asks your browser to accept a hidden `200x200` canvas.
2.  It asks to draw: "Text with Emoji", "3D Gradient", "Complex Geometry".
3.  Each GPU (Nvidia 3080 vs Intel Iris vs Apple M1) renders anti-aliasing slightly differently.
4.  The website converts the image to a Hash (`f821cc...`).
5.  This Hash is your device ID. It survives clearing cookies.

## 2. The Problem with "Headless"
If you run Headless Chrome on Linux (VPS), the "Monitor" is missing.
The rendering is done by "SwiftShader" (Software Rendering).
*   **Result:** Unique ID = "100% Bot".
*   Real users have hardware GPUs.

## 3. Defense: Noise Injection
You cannot "fake" an Nvidia 3080 easily.
But you can **Poison** the hash.
If you change just **1 pixel** RGB value by **1 bit** (e.g., color 255 -> 254), the entire Hash changes.
If you randomize this noise every session, you look like a new device every time.

## 4. Implementation (Code)
Using `selenium-stealth` or similar injection scripts.

```python
from selenium import webdriver
from selenium_stealth import stealth

options = webdriver.ChromeOptions()
options.add_argument("start-maximized")
driver = webdriver.Chrome(options=options)

stealth(driver,
    languages=["en-US", "en"],
    vendor="Google Inc.",
    platform="Win32",
    webgl_vendor="Intel Inc.",
    renderer="Intel Iris OpenGL Engine",
    fix_hairline=True,
)

driver.get("https://browserleaks.com/canvas")
# Check the "Signature".
# Run script again. Check signature. It should be different (or same, depending on config).
```

## 5. Homework
1.  Go to `browserleaks.com/canvas` with your normal Chrome. Note the Signature.
2.  Go there with Incognito. Note the Signature. (It is the SAME! Incognito does not hide hardware).
3.  Write a script to visit it and confirm you can get a different signature.
