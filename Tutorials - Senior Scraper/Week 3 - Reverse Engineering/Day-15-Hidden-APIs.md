# Day 15: The Hidden Door (Traffic Interception)

**Objective:** Stop parsing HTML (BeautifulSoup). Start parsing JSON (APIs).
**Tools:** Chrome DevTools, Postman/Insomnia.

---

## 1. The Theory: SPA (Single Page Applications)
Modern sites (React, Vue, Angular) are empty shells.
1.  Browser loads `index.html` (Empty Divs).
2.  Browser saves `bundle.js`.
3.  **JavaScript** makes a background request to `/api/v1/products`.
4.  Server returns **JSON**.
5.  JavaScript builds the HTML.

**The Junior Scraper** waits for step 5 and parses HTML.
**The Senior Scraper** finds step 3 and gets the raw data.

## 2. The Technique: "Look for the Elephant"
When you load a page with 100 products, the computer MUST receive the text for those 100 products.
It's hidden in the network traffic.

**Exercise:**
1.  Open Chrome DevTools (`F12`).
2.  Go to the **Network** tab.
3.  Click the **Fetch/XHR** filter (This hides images/css).
4.  Go to `instagram.com/nike` (or any dynamic site).
5.  Refresh the page.
6.  Look at the list. Click on items until you see the **Preview** tab showing the data you see on screen (e.g., follower count, post text).

## 3. "Copy as cURL"
Once you find the request (e.g., `web_profile_info`), you need to replicate it in Python.

1.  Right Click the request in DevTools.
2.  `Copy` -> `Copy as cURL (bash)`.
3.  Go to a converter like [curlconverter.com](https://curlconverter.com/).
4.  Paste the cURL.
5.  It generates Python `requests` code with ALL headers/cookies.

## 4. Automation Pattern
You can't manually copy cURL for every request. You need to find the **Pattern**.

*   Look at the URL: `https://api.site.com/users/12345/posts`
*   Change `12345` to another user ID. Does it work?
*   If **yes**, you have found an OPEN API.
*   If **no** (403 Forbidden), the site needs a **Signature** (Day 19).

## 5. Homework
1.  Go to `https://www.airbnb.com/`.
2.  Search for "London".
3.  Find the XHR request that contains the search results (Price, Title, etc).
4.  Copy it to Python and verify you can get the JSON data without using Selenium.
