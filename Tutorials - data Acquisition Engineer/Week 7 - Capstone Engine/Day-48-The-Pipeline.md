# Day 48: The Refinery (The Consumer)

**Objective:** Clean the raw flight data and store it.
**Stack:** Kafka Consumer + Pydantic + Postgres.

---

## 1. The Schema
We need to standardize the messy JSON from the API.

```python
from pydantic import BaseModel, validator
from datetime import datetime

class Flight(BaseModel):
    airline: str
    price: float
    currency: str
    departure_time: datetime
    arrival_time: datetime
    
    @validator('currency')
    def normalize_currency(cls, v):
        return v.upper() # Ensure "usd" becomes "USD"
```

## 2. The Consumer Logic

```python
import json
from kafka import KafkaConsumer
import psycopg2

# 1. DB Connection
conn = psycopg2.connect("dbname=flights user=postgres password=secret")
cur = conn.cursor()

# 2. Setup Table
cur.execute("""
    CREATE TABLE IF NOT EXISTS flights (
        id SERIAL PRIMARY KEY,
        airline VARCHAR(50),
        price DECIMAL(10, 2),
        currency VARCHAR(3),
        dept_time TIMESTAMP,
        arr_time TIMESTAMP
    )
""")
conn.commit()

# 3. Consumer
consumer = KafkaConsumer(
    'raw_flights',
    bootstrap_servers=['localhost:9092'],
    value_deserializer=lambda x: json.loads(x.decode('utf-8')),
    group_id='flight_archiver'
)

print("💾 Archiver Running...")

for msg in consumer:
    raw_data = msg.value
    try:
        # Validate
        clean_flight = Flight(**raw_data)
        
        # Insert
        cur.execute("""
            INSERT INTO flights (airline, price, currency, dept_time, arr_time)
            VALUES (%s, %s, %s, %s, %s)
        """, (
            clean_flight.airline, 
            clean_flight.price, 
            clean_flight.currency, 
            clean_flight.departure_time, 
            clean_flight.arrival_time
        ))
        conn.commit()
        print(f"✅ Saved flight: {clean_flight.airline} ${clean_flight.price}")
        
    except Exception as e:
        print(f"❌ Error: {e}")
        # Send to DLQ
```

## 3. Graduation
You have the Engine running.
*   Scraper (Producer) -> Kafka -> Consumer -> DB.
*   It handles polling.
*   It acts like Chrome.
*   It validates data.

**Next Week:** Deployment.
We wrap this all in Docker and deploy it to a Cloud VPS.
