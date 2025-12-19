# Day 25: Reading the Matrix II (Mobile Analysis)

**Objective:** Find the JSON Endpoint in MITMProxy and convert it to Python.
**Tools:** MITMProxy ("Flow" view).

---

## 1. The Noise
Mobile apps are noisy. They send analytics (`firebase`, `crashlytics`, `facebook_graph`) every second.
If you open MITMProxy, you will see 100s of requests.

**Filtering:**
In the MITMWeb search bar, type:
`~t json & !~u google & !~u facebook`
*   `~t json`: Only show requests with content-type JSON.
*   `!~u google`: Exclude URLs containing "google".

## 2. Triggering the Event
1.  Clear the MITM list (Press `z` or click Clear).
2.  On the App, perform the **EXACT** action you want to scrape (e.g., Search for "Pizza").
3.  Look at the new requests.
4.  One of them will be the "Golden Endpoint".

## 3. The "Golden Endpoint"
It usually looks like: `https://api.target.com/v2/search?q=Pizza`.
Click it.
Go to the **Response** tab.
Do you see "Pizza" in the JSON? **Yes.**
You found it.

## 4. Exporting to Python
MITMProxy has a "Copy as cURL" feature, but sometimes it's hidden in the flow menu.

1.  Right click the request.
2.  `Copy` -> `cURL`.
3.  Go to [curlconverter.com](https://curlconverter.com).
4.  Paste.
5.  **Clean up:**
    Mobile headers are HUGE.
    *   `X-Device-Id`: Might be needed.
    *   `X-App-Version`: Might be needed.
    *   `Authorization`: Definitely needed.
    *   `User-Agent`: (Dalvik/2.1.0...) Keep this.

## 5. Homework
1.  Intercept the **Search** function of a Food Delivery App (UberEats / DoorDash / local equiv).
2.  Find the JSON that lists the restaurants.
3.  Write a Python script that searches for "Sushi" and prints the first 5 restaurant names.
