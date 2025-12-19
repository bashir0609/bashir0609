# Day 46: The Engine (The Producer)

**Objective:** Write the script that fetches flights and sends them to the Buffer.
**Stack:** Python + curl_cffi + Kafka.

---

## 1. The Class Structure

```python
import time
import json
from curl_cffi import requests
from kafka import KafkaProducer

class FlightScraper:
    def __init__(self):
        self.producer = KafkaProducer(
            bootstrap_servers=['localhost:9092'],
            value_serializer=lambda v: json.dumps(v).encode('utf-8')
        )
        # Use Chrome 124 Impersonation to pass Cloudflare
        self.session = requests.Session(impersonate="chrome124")

    def create_session(self, origin, dest, date):
        print(f"🔎 Searching {origin} -> {dest} on {date}")
        url = "https://api.target.com/v1/flights/search"
        payload = {"from": origin, "to": dest, "date": date}
        
        resp = self.session.post(url, json=payload)
        return resp.json()['session_id']

    def poll_results(self, session_id):
        url = f"https://api.target.com/v1/flights/poll/{session_id}"
        
        while True:
            resp = self.session.get(url)
            data = resp.json()
            
            status = data['status']
            print(f"Status: {status}")
            
            if status == "COMPLETE":
                return data['flights']
            
            time.sleep(2) # Polling delay

    def run(self, origin, dest, date):
        sess_id = self.create_session(origin, dest, date)
        flights = self.poll_results(sess_id)
        
        print(f"🚀 Sending {len(flights)} flights to Kafka...")
        for f in flights:
            self.producer.send('raw_flights', f)

if __name__ == "__main__":
    bot = FlightScraper()
    bot.run("LHR", "JFK", "2024-12-25")
```

## 2. Key Concepts
1.  **Impersonation:** We use `curl_cffi` because major travel sites act like "Hard Targets".
2.  **State Management:** We keep the same `self.session` so cookies persist between the POST and the Poll.
3.  **Buffering:** We don't save to DB. We just fire to Kafka. The scraping is decoupled from storage.

## 3. Homework
1.  Implement this for your chosen target.
2.  Verify data appears in the Redpanda console.
