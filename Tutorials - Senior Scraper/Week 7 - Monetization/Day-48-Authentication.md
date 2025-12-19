# Day 48: The Bouncer (Auth & Rate Limits)

**Objective:** Prevent people from using your API for free.
**Tools:** HTTPBearer, SlowAPI (Rate Limiter).

---

## 1. API Keys (The Simple Way)
We don't need complex OAuth. We just need a secret key.

```python
from fastapi import Security, HTTPException, Depends
from fastapi.security.api_key import APIKeyHeader

API_KEY_NAME = "X-API-Key"
api_key_header = APIKeyHeader(name=API_KEY_NAME, auto_error=False)

# In a real app, this comes from a database
VALID_KEYS = {
    "sk_live_12345": {"user": "islah", "plan": "premium"},
    "sk_live_67890": {"user": "guest", "plan": "free"}
}

async def get_api_key(api_key_header: str = Security(api_key_header)):
    if api_key_header in VALID_KEYS:
        return VALID_KEYS[api_key_header]
    raise HTTPException(status_code=403, detail="Invalid API Key")

@app.get("/premium-data")
async def sensitive_endpoint(user: dict = Depends(get_api_key)):
    return {"data": "Secret info", "user": user}
```

## 2. Rate Limiting (Throttling)
You don't want a user spamming 100 requests/second.
Tool: `slowapi` (Extension for FastAPI).

```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

@app.get("/scrape")
@limiter.limit("5/minute") # Limit to 5 requests per minute per IP
async def scrape(request: Request):
    return {"status": "ok"}
```

## 3. The Business Model
*   **Free:** 10 requests / day (Slow execution).
*   **Pro ($29/mo):** 10,000 requests / day (Priority Queue).
*   **Enterprise:** Unlimited.

## 4. Homework
1.  Combine Day 43 (API), Day 46 (Queue), and Day 48 (Auth).
2.  Create a `server.py` that checks the API Key. if valid, pushes the Job to Redis.
3.  Deploy it. (Week 8).
