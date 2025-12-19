# Day 17: Reading the Matrix (AST Parsing)

**Objective:** Extract a variable from a `.js` file without running a browser.
**Concept:** Abstract Syntax Tree (AST).

---

## 1. The Scenario
You are scraping a site.
The data is inside a `<script>` tag:
```html
<script>
    window.__PRELOADED_STATE__ = {"user": "islah", "token": "1a2b3c..."};
</script>
```
You could use Regex (`re.search`), but Regex is fragile. If the JS adds a newline, Regex breaks.
Use **Parser Logic**.

## 2. Tools
*   **Simple:** `chompjs` (Recommended). It turns JS Objects into Python Dicts.
*   **Complex:** `slimit` or `esprima` (Full AST traversal).

## 3. Using ChompJS
This library is magic. It parses minimal JS into valid JSON.

```bash
pip install chompjs
```

```python
import chompjs
from bs4 import BeautifulSoup

html = """
<html>
<body>
<script>
    var config = {
        apiKey: "12345",
        features: ["a", "b"]
    };
</script>
</body>
</html>
"""

soup = BeautifulSoup(html, "html.parser")
script_content = soup.find("script").string

# Extract just the object part
# (You might need a tiny regex to find the start of the object)
import re
raw_obj = re.search(r'var config = ({.*?});', script_content, re.DOTALL).group(1)

data = chompjs.parse_js_object(raw_obj)
print(data['apiKey']) # 12345
```

## 4. Why Extract?
Web sites often hardcode:
*   **API Keys** (`apiKey: "..."`).
*   **App IDs** (`appId: "..."`).
*   **Initial Data** (`__NEXT_DATA__`).

If you can extract these from the HTML/JS source, you can then make direct API calls (Day 15/16) without ever loading the full page functionality.

## 5. Homework
1.  Go to a Next.js website (e.g., Target.com, Nike.com).
2.  View Source.
3.  Search for `__NEXT_DATA__`.
4.  Write a script to extract that JSON blob and parse it with `json.loads` (since Next.js data is usually valid JSON).
