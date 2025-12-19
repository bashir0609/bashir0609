# Day 31: The Drain (Kafka Consumer)

**Objective:** Read data from the buffer and save it safely.
**Component:** The Consumer.

---

## 1. The Power of "Offset"
Kafka remembers where you left off.
If you read 50 items and then Crash, when you restart, Kafka gives you item 51.
This is called **Offset Commit**.

## 2. The Python Consumer
Create `consumer.py`:

```python
import json
from kafka import KafkaConsumer
import time

# 1. Connect
consumer = KafkaConsumer(
    'scraped_items', # Topic Name
    bootstrap_servers=['localhost:9092'],
    auto_offset_reset='earliest', # Start from beginning if new
    enable_auto_commit=True,
    group_id='db_writer_group', # Critical: Workers with same Group share the load
    value_deserializer=lambda x: json.loads(x.decode('utf-8'))
)

def save_to_db(item):
    # Simulate DB Write
    print(f"💾 Saving to Postgres: {item['url']} - ${item['price']}")
    time.sleep(0.1) # Simulate Latency

if __name__ == "__main__":
    print("📡 Waiting for data...")
    for message in consumer:
        item = message.value
        save_to_db(item)
```

## 3. The Experiment
1.  Keep `consumer.py` running.
2.  Start `producer.py` (Day 30) in another terminal.
3.  Watch the data flow.
    *   **Scraper:** "Sent Item 50"
    *   **Consumer:** "Saved Item 50"

**Now test resilience:**
1.  Stop the `consumer.py`.
2.  Keep `producer.py` running for 10 seconds. (It buffers 10 items).
3.  Start `consumer.py` again.
4.  **Watch it catch up!** It will rapidly print the missed items.

## 4. Homework
1.  Get the Consumer running.
2.  Perform the "Crash Test" above.
3.  Verify no data was lost.
