# Day 39: The Endless Pit (Infinite Scroll & Virtualization)

**Objective:** Scrape 1000 posts from an Instagram Profile.
**Problem:** The HTML only shows 12 posts. When you scroll, the old ones are **deleted** from the DOM (Virtualization).

---

## 1. The React Virtual DOM
Social sites (FB, IG, Twitter) use "Virtual Scrollers".
*   They only render what is on the screen to save memory.
*   If you scroll down 50 times, the top posts exist in Javascript memory but NOT in the HTML `<div>`.
*   **Result:** `soup.find_all('div')` only returns the last 12 posts.

## 2. Solution A: The Network Intercept (Preferred)
Don't fight the DOM. Fight the Network.
When you scroll, the page sends a request to `graphql/query`.
**Intercept THAT request** (Day 15 skill).

## 3. Solution B: The "Playwright List" (If you must use Browser)
If you are forced to use a browser (e.g., due to strict signature signing), you need a loop that:
1.  Scrapes visible items.
2.  Adds them to a Python Set (to handle duplicates).
3.  Scrolls slightly.
4.  Repeats.

## 4. Instagram Logic (Network Approach)
Instagram uses GraphQL.
1.  **Query Hash:** `69cba4...` (Static ID for "Get User Posts").
2.  **Variables:** `{"id":"123","first":12,"after":"cursor_xyz"}`.
    *   `after`: The "Pagination Cursor" (End Cursor).

**The Loop:**
1.  Request Page 1.
2.  Get `edges` (posts) and `page_info.end_cursor`.
3.  Request Page 2 with `after=page_info.end_cursor`.
4.  Repeat until `has_next_page=false`.

## 5. The Code (Python Logic)

```python
# Pseudo-code for pagination loop
cursor = ""
while True:
    variables = {
        "id": "USER_ID_HERE",
        "first": 50,
        "after": cursor
    }
    
    # You need the Session cookies + Headers from Day 10/21
    resp = session.get(IG_GRAPHQL_URL, params={"variables": json.dumps(variables)})
    data = resp.json()
    
    posts = data['data']['user']['edge_owner_to_timeline_media']['edges']
    for post in posts:
        print(post['node']['display_url'])
        
    page_info = data['data']['user']['edge_owner_to_timeline_media']['page_info']
    if not page_info['has_next_page']:
        break
        
    cursor = page_info['end_cursor']
    time.sleep(random.uniform(2, 5)) # Sleep to avoid Rate Limit
```

## 6. Homework
1.  Log in to Instagram on Desktop Chrome.
2.  Open DevTools -> Network -> Fetch/XHR.
3.  Scroll down the profile.
4.  Find the Request `?query_hash=...`.
5.  Look at the Response. Find the `end_cursor`.
