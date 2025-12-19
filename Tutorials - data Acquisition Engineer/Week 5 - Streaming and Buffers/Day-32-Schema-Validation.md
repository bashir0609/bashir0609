# Day 32: The Guard (Schema Validation)

**Objective:** Prevent "Garbage In, Sewer Out".
**Concept:** Schema Registry / Dead Letter Queue.

---

## 1. The Horror Story
You scrape Amazon.
Amazon changes the HTML.
Your scraper now sends `price: None` or `price: "Call for details"`.
Your Consumer blindly inserts this into the Database.
**Result:** Currently, your dashboard shows "Average Price: NaN". You just corrupted 1 million rows.

## 2. Pydantic to the Rescue
We must Validate the data **twice**.
1.  **Producer Side:** Don't send trash to Kafka.
2.  **Consumer Side:** Don't write trash to Postgres.

## 3. Implementation
Use `pydantic` models.

```python
from pydantic import BaseModel, ValidationError, validator

class Product(BaseModel):
    url: str
    price: float
    status: str

    @validator('price')
    def price_must_be_positive(cls, v):
        if v <= 0:
            raise ValueError('Price implies free product, unlikely.')
        return v
```

**Updated Consumer:**

```python
def save_to_db(raw_dict):
    try:
        # Validate
        item = Product(**raw_dict)
        
        # If valid, Save
        print(f"✅ Saved ${item.price}")
        
    except ValidationError as e:
        # 4. Dead Letter Queue logic
        print(f"❌ BAD DATA REJECTED: {e}")
        # Option: Send to a separate 'errors' topic
        # producer.send('dead_letter_queue', raw_dict)
```

## 4. The Dead Letter Queue (DLQ)
If data is bad, **DO NOT CRASH**.
Do not stop the consumer.
Just create a separate Kafka topic called `scraping_errors`.
Send the bad JSON there.
Later, you can inspect that topic to see *why* scraping is failing (e.g., "Ah, Amazon changed the class name").

## 5. Homework
1.  Modify your Producer to send a "bad" item (e.g., price = -5).
2.  Modify your Consumer to use Pydantic.
3.  Verify that the Consumer prints "REJECTED" but keeps running for the next valid item.
