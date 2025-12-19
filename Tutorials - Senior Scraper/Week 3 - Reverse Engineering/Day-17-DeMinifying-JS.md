# Day 17: Reading the Matrix (De-Minifying JS)

**Objective:** Find API Keys and Logic hidden in the `app.js` bundle.
**Tools:** Chrome DevTools (Sources Tab), "Prettier".

---

## 1. What is Minification?
Developers write code like this:
```javascript
function getPrice(productId) {
    return api.call("/price/" + productId, "SECRET_KEY_123");
}
```

Webpack turns it into this (to save space):
```javascript
function a(b){return c.d("/price/"+b,"SECRET_KEY_123")}
```

Your job is to read the second version to find `"SECRET_KEY_123"`.

## 2. Searching Global Variables
Many sites accidentally expose config variables globally.
1.  Open DevTools Console.
2.  Type `window.` and look at the autocomplete.
3.  Look for:
    *   `window.__INITIAL_STATE__` (React Redux state).
    *   `window.__NEXT_DATA__` (Next.js data - Goldmine!).
    *   `window.config`
    *   `window.context`

**Example (Next.js sites):**
If you go to a Nike/Ticketmaster page, view source and search for `__NEXT_DATA__`.
It’s a giant JSON string containing all the data on the page. You don't need to parse HTML classes!

## 3. Finding the "Signer"
If an API requires a header like `X-Signature: abcd123...`, you need to find **where** it is generated.

1.  Go to the **network tab**. Find the request with the signature.
2.  Hover over the **Initiator** column (e.g., `bundle.js:14502`).
3.  Click it. Chrome will take you to the line of code that made the request.
4.  It will look ugly (minified). Click the **{} (Pretty Print)** button in the bottom left of the Sources panel.
5.  Now you can read the function. Use `Ctrl+F` to search for "Signature" or the specific header name.

## 4. Breakpoints (The Interactive Way)
You found the function `f(x)`. You want to know what it does.
1.  Click the line number (e.g., 2045) to set a blue "Breakpoint".
2.  Trigger the action on the page (e.g., click specific button).
3.  The browser **PAUSES** execution.
4.  Hover over variables `a`, `b`, `c` to see their real values (e.g., `a` = "POST", `b` = "/api/login").

## 5. Homework
1.  Go to `twitter.com`.
2.  Search in sources for `Bearer`.
3.  Find the "Guest Token" or "Authorization" header value in the Javascript.
