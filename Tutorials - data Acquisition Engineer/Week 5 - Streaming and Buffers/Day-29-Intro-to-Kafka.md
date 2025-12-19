# Day 29: The Firehose (Intro to Kafka)

**Objective:** Understand why "Scraper -> Database" is a bad architecture.
**Concept:** Decoupling & Buffering.

---

## 1. The Junior Architecture
`Scraper.py` -> `INSERT INTO Postgres`.

**Why it fails:**
1.  **Speed Mismatch:** You scrape 100 pages/sec (fast). Postgres writes 50 rows/sec (slow).
2.  **Locks:** If Postgres locks a table for maintenance, your Scraper crashes.
3.  **Data Loss:** If Scraper crashes mid-batch, data is lost.

## 2. The Senior Architecture
`Scraper.py` -> **Kafka (Buffer)** -> `Consumer.py` -> `Postgres`.

**Why it wins:**
1.  **Scraper doesn't care:** It fires data into Kafka at 1000 msg/sec and disconnects. Fast.
2.  **Safety:** Kafka persists data to disk instantly. If Postgres dies, data waits in Kafka for days.
3.  **Scale:** You can start 10 Consumers to read from Kafka if the buffer gets full.

## 3. What is Kafka?
Think of it as a **Log File**.
1.  **Producer (Scraper):** Appends a line to the log.
2.  **Consumer (Writer):** Reads the log line by line.

## 4. Why not Redis?
We used Redis queues in Week 7 (Senior Scraper Path).
*   **Redis:** Good for "Jobs" (Do this task). In-memory (can lose data).
*   **Kafka:** Good for "Data Events" (Here is a product). On-disk (persistent, replayable).

## 5. Homework
1.  Read about "Pub/Sub" architecture.
2.  Install Docker Desktop (if not already installed). We need it for Day 30.
