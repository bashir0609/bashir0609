# Day 30: The Pump (Kafka Producer)

**Objective:** Start Kafka and send data to it.
**Tool:** Redpanda (It's Kafka, but lighter and no JVM required).

---

## 1. Structure
Folder: `kafka_scraper`
*   `docker-compose.yml`
*   `producer.py`

## 2. Docker Setup (Redpanda)
Create `docker-compose.yml`:
```yaml
version: "3.7"
services:
  redpanda:
    image: docker.redpanda.com/redpanda/redpanda:latest
    ports:
      - 8081:8081
      - 9092:9092
    command:
      - redpanda
      - start
      - --overprovisioned
      - --smp 1
      - --memory 1G
      - --reserve-memory 0M
      - --node-id 0
      - --check=false
    container_name: redpanda
```
Run: `docker-compose up -d`.

## 3. The Python Producer
Install library: `pip install kafka-python-ng` (or `confluent-kafka`).

```python
import json
import time
import random
from kafka import KafkaProducer

# 1. Connect to Redpanda (Kafka)
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

def scrape_page():
    # Simulate scraping
    return {
        "url": f"https://example.com/item/{random.randint(1, 1000)}",
        "price": random.randint(10, 100),
        "status": "active"
    }

if __name__ == "__main__":
    print("🚀 Scraper started...")
    while True:
        data = scrape_page()
        
        # 2. Send to "Buffer" (Topic: 'scraped_items')
        producer.send('scraped_items', data)
        print(f"Sent: {data}")
        
        # In real life, don't sleep. Go as fast as possible.
        time.sleep(1) 
```

## 4. Why this is powerful
Run the script.
Now stop the script.
The data is SAFTELY stored in Redpanda. Even if you don't read it for 5 hours, it's there.
This is **Persistence**.

## 5. Homework
1.  Run Docker.
2.  Run the Producer.
3.  Let it send 100 items.
4.  Stop the Producer.
