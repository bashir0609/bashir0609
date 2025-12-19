# Day 11: The Sound (AudioContext Fingerprinting)

**Objective:** Hide your Audio Hardware signature.
**Concept:** AudioContext API.

---

## 1. How it works
1.  Website creates an `Oscillator` (Sound Generator) in the background.
2.  It plays a tiny sine wave.
3.  It asks the browser to "Calculate the specific frequency results".
4.  Your Audio Driver (Realtek, Nvidia HDMI, etc.) has tiny mathematical floating-point variances.
5.  Result: A unique float like `124.043943...`.
6.  This is your Audio Fingerprint.

## 2. The Danger
Just like Canvas, if you run on an AWS Server (Headless), you have **NO Sound Card**.
The API returns either `0` or a "Null Audio Device" signature.
This is a 100% guarantee that you are a Bot/VPS.

## 3. The Defense
We must inject JavaScript to spoof the return values of the `AudioContext` functions.
We basically lie about the math.

## 4. Code (Injection)
Most privacy extensions (like "AudioContext Fingerprint Defender") use a script like this:

```javascript
// Injected via Driver
const originalGetChannelData = AudioBuffer.prototype.getChannelData;
AudioBuffer.prototype.getChannelData = function() {
    const results = originalGetChannelData.apply(this, arguments);
    for (let i = 0; i < results.length; i += 100) {
        // Add minimal noise
        results[i] += 0.000001; 
    }
    return results;
}
```

In Python/Selenium, you load this JS and execute it via `driver.execute_cdp_cmd` "Page.addScriptToEvaluateOnNewDocument".

## 5. Homework
1.  Visit [audiofingerprint.openwpm.com](https://audiofingerprint.openwpm.com/).
2.  Run your bot.
3.  Check if it successfully generates a fingerprint or if it errors out (common in Headless).
4.  If it errors, you need to enable `--use-fake-device-for-media-stream` flags in ChromeOptions.
