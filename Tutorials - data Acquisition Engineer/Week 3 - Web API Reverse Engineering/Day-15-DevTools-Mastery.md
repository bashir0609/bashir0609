# Day 15: The X-Ray (DevTools & Postman)

**Objective:** Stop parsing HTML. Start parsing API responses.
**Concept:** "Hidden" JSON APIs.

---

## 1. The Single Page App (SPA) Revolution
Modern sites (React/Vue/Next.js) do not send the data in the initial HTML.
1.  Browser loads `index.html` (Empty Shell).
2.  Browser fetches `bundle.js`.
3.  **JavaScript fetches `/api/v1/products` (The Gold Mine).**
4.  JavaScript renders the products.

**The Mistake:** Scrapers parse step 4.
**The Pro:** Scrapers intercept step 3.

## 2. DevTools Strategy
1.  Open DevTools (`F12`) -> **Network** Tab.
2.  Select **Fetch/XHR** filter (Ignore CSS/Images).
3.  Refresh the page.
4.  Look for the request that contains the data you see on screen.
    *   *Tip:* Use `Ctrl+F` in the network tab to search for a specific product name.

## 3. "Copy as cURL"
Once you find the request:
1.  Right Click -> `Copy` -> `Copy as cURL (bash)`.
2.  Go to [curlconverter.com](https://curlconverter.com).
3.  Paste it.
4.  Get Python code.

## 4. Replay in Postman
Before writing code, verify the API is open.
1.  Import the cURL into Postman.
2.  Send.
3.  **Delete Headers one by one.**
    *   Delete `Cookie` -> Does it still work? (If yes, it's public data!)
    *   Delete `Referer` -> Does it still work?
    *   Delete `Authorization` -> Does it still work?
4.  Find the **Minimum Viable Request**.

## 5. Homework
1.  Go to a dynamic site (e.g., Airbnb or Instagram).
2.  Find the JSON response for the search results.
3.  Replicate that request in Python `requests` with the minimal headers needed.
