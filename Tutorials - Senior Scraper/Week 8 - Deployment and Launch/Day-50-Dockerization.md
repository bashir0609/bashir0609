# Day 50: The Shipping Container (Docker)

**Objective:** "It works on my machine" -> "It works everywhere."
**Tools:** Docker, Docker Compose.

---

## 1. The Dockerfile
Pass this file to anyone, and they can run your scraper without installing Python.

**File:** `Dockerfile`
```dockerfile
# Start with a lightweight Python image
FROM python:3.11-slim

# Install system dependencies (needed for Playwright browsers)
# Note: For playwright, you usually use the official playwright image
# FROM mcr.microsoft.com/playwright/python:v1.43.0-jammy

WORKDIR /app

# Copy requirements first (for caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy logic
COPY . .

# Default command
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 2. Docker Compose (The Orchestrator)
You have 3 moving parts now:
1.  **API** (FastAPI).
2.  **Worker** (Arq).
3.  **Database** (Redis).

**File:** `docker-compose.yml`
```yaml
version: '3.8'

services:
  # 1. The Brain (Redis)
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

  # 2. The API (FastAPI)
  api:
    build: .
    command: uvicorn main_async:app --host 0.0.0.0 --port 8000
    ports:
      - "8000:8000"
    depends_on:
      - redis
    environment:
      - REDIS_HOST=redis

  # 3. The Muscle (Worker)
  worker:
    build: .
    command: arq worker.WorkerSettings
    depends_on:
      - redis
    environment:
      - REDIS_HOST=redis
```

## 3. Running it
1.  `docker-compose up --build`
2.  Wait for green text.
3.  Go to `localhost:8000`.

**Congratulations.** You just built a Microservices Architecture.
You can now scale up workers: `docker-compose up --scale worker=5`.
Now you have 5 scrapers running in parallel.

## 4. Homework
1.  Install Docker Desktop.
2.  Create the files above.
3.  Run `docker-compose up`.
4.  Verify you can hit the API and the Worker logs the job.
