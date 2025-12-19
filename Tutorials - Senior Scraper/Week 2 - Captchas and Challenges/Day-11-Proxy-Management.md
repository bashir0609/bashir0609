# Day 11: The Mask (Proxy Management)

**Objective:** Understand why your IP gets banned and how to rotate 1,000,000 IPs.
**Cost:** You assume proxies are expensive. Bad proxies are cheap. Good proxies cost money.

---

## 1. Proxy Types Hierarchy

| Tier | Name | Source | Use Case | Detectability | Cost |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | **Datacenter (DC)** | AWS, DigitalOcean IPs | Simple sites (Wikipedia) | High (Often blocked by subnet) | Cheap ($1/IP) |
| 2 | **ISP / Static Resi** | Comcast/Verizon IPs owned by a host | Sneaker bots, Account management | Medium | Moderate |
| 3 | **Residential** | Grandma's WiFi (P2P networks) | High-security scraping | Low (Looks like a real user) | Expensive ($10/GB) |
| 4 | **Mobile (4G/5G)** | Real SIM cards | Creating Instagram Accounts | Zero (IP changes every minute) | Very Expensive |

**Rule of Thumb:** Start with DC. If blocked, move to Residential.

## 2. Authentication Styles

**A. User:Pass (The Standard)**
`http://user:pass@192.168.1.1:8000`
Static IP. Good for "I need this specific IP".

**B. Backconnect (The Rotator)**
`http://user:pass@gateway.proxyprovider.com:8000`
You connect to ONE gateway. *They* route you to a different exit node every request.
**This is best for scraping.** You don't manage a list of IPs; you just hit the gateway.

## 3. Implementation: The Rotator

If you buy a list of 1000 DC proxies, you need to rotate them yourself.

```python
import requests
import random
import itertools

# Your list of proxies
PROXY_LIST = [
    "http://user:pass@1.1.1.1:8000",
    "http://user:pass@2.2.2.2:8000",
    "http://user:pass@3.3.3.3:8000",
]

# Option 1: Random Choice (Good for stateless requests)
def get_random_proxy():
    return random.choice(PROXY_LIST)

# Option 2: Round Robin (Good for distributing load evenly)
proxy_cycle = itertools.cycle(PROXY_LIST)
def get_next_proxy():
    return next(proxy_cycle)

def scrape(url):
    proxy_url = get_random_proxy()
    proxies = {
        "http": proxy_url,
        "https": proxy_url
    }
    
    try:
        print(f"Using proxy: {proxy_url[-10:]}")
        r = requests.get(url, proxies=proxies, timeout=5)
        print(f"Status: {r.status_code}")
    except Exception as e:
        print(f"Proxy failed: {e}")
        # Logic to remove bad proxy from list?

if __name__ == "__main__":
    for _ in range(5):
        scrape("https://httpbin.org/ip")
```

## 4. Middleware Pattern
In Scrapy or large projects, we don't code this in every function. We use "Middleware".
*   Request starts.
*   Middleware attaches Proxy.
*   Response comes back.
*   If status == 403 or 429 (Banned):
    *   Middleware catches exception.
    *   Mark proxy as "Dead".
    *   Retries request with NEW proxy.

## 5. Homework
1.  Find a "Free Proxy List" online (they are terrible, but good for testing code).
2.  Write a script that checks 10 of them against `httpbin.org/ip`.
3.  Calculate your "Success Rate" (e.g. 2/10 worked).
