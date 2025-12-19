# Day 46: The Queue (Async Architecture)

**Objective:** Handle a request that takes 5 minutes to scrape without the API timing out.
**Problem:** If you call `time.sleep(60)` in FastAPI, the user hangs. If they close the tab, the script dies.

---

## 1. The Solution: Job ID Pattern
1.  **User:** `POST /scrape`
2.  **API:** "I received your order. Here is your Ticket Number: **JOB_123**. Bye." (Returns instantly).
3.  **Worker:** Sees **JOB_123** in Redis. Starts scraping in background.
4.  **User:** `GET /status/JOB_123` -> "Processing..."
5.  **User:** `GET /status/JOB_123` -> "Completed! Here is data."

## 2. Tools
*   **Redis:** The database for the queue. (Install via Docker or WSL).
*   **Arq / Celery:** The worker library. We will use `arq` (Modern, Async).

## 3. The Code

**file: worker.py**
```python
from arq import create_pool
from arq.connections import RedisSettings
import asyncio

async def scrape_task(ctx, url: str):
    print(f"👷 Worker starting on {url}...")
    await asyncio.sleep(10) # Simulate long scraping (Playwright etc)
    print("✅ Finished!")
    return {"title": "Scraped Title", "url": url}

async def startup(ctx):
    print("Worker starting up...")

class WorkerSettings:
    functions = [scrape_task]
    on_startup = startup
    redis_settings = RedisSettings()
```

**file: main_async.py (FastAPI)**
```python
from fastapi import FastAPI
from arq import create_pool
from arq.connections import RedisSettings

app = FastAPI()

@app.post("/scrape")
async def scrape_endpoint(url: str):
    redis = await create_pool(RedisSettings())
    
    # Enqueue the job
    job = await redis.enqueue_job('scrape_task', url)
    
    return {"job_id": job.job_id, "status": "queued"}

@app.get("/status/{job_id}")
async def status_endpoint(job_id: str):
    redis = await create_pool(RedisSettings())
    job = redis.job(job_id) # Get job handle
    
    status = await job.status()
    info = await job.info()
    
    response = {"status": str(status)}
    
    if status == "complete":
        response["result"] = await job.result()
        
    return response
```

## 4. Running it
1.  Start Redis: `docker run -p 6379:6379 redis`
2.  Start Worker: `arq worker`
3.  Start API: `uvicorn main_async:app`
4.  Hit the API. You get a Job ID. The Worker logs "Starting...".

This is how **Every Enterprise Scraper** works.

## 5. Homework
1.  Install Redis (If on Windows, use Memurai or WSL).
2.  Get the `arq` example running.
3.  Make the worker run your Day 9 (Captcha Solver) script.
