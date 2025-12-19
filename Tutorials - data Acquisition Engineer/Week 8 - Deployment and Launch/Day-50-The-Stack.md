# Day 50: The Stack (Docker Compose)

**Objective:** Run your entire company with one command.
**Concept:** Orchestration.

---

## 1. The Components
We have built:
1.  **Scraper (Producer)**
2.  **Kafka (Redpanda)**
3.  **Postgres (Storage)**
4.  **Dashboard (Streamlit - Day 53)**

## 2. The `docker-compose.yml`
This is your masterpiece.

```yaml
version: '3.8'

services:
  # 1. The Buffer
  redpanda:
    image: docker.redpanda.com/redpanda/redpanda:latest
    ports: ["9092:9092"]
    command: ["redpanda", "start", "--overprovisioned", "--smp", "1", "--memory", "1G"]

  # 2. The Storage
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: flights
    ports: ["5432:5432"]
    volumes:
      - pg_data:/var/lib/postgresql/data

  # 3. The Scraper (Producer)
  scraper:
    build: ./scraper
    command: python producer.py
    depends_on:
      - redpanda
    environment:
      BOOTSTRAP_SERVER: redpanda:9092

  # 4. The Consumer (Archiver)
  consumer:
    build: ./consumer
    command: python consumer.py
    depends_on:
      - redpanda
      - postgres
    environment:
      BOOTSTRAP_SERVER: redpanda:9092
      DB_HOST: postgres

volumes:
  pg_data:
```

## 3. Configuration
Notice we use `environment` variables.
In your Python code, you must change:
```python
# OLD
bootstrap_servers=['localhost:9092']

# NEW
import os
server = os.getenv('BOOTSTRAP_SERVER', 'localhost:9092')
bootstrap_servers=[server]
```

## 4. Homework
1.  Create the `docker-compose.yml`.
2.  Create `Dockerfile` for your scraper and consumer.
3.  Run `docker-compose up --build`.
4.  Watch the logs mix together. PRODUCER -> KAFKA -> CONSUMER -> DB.
