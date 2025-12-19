# Day 33: The Scale (Consumer Groups)

**Objective:** Handle 10,000 items/second.
**Concept:** Parallelism.

---

## 1. The Problem
Your `producer` is fast. It writes 1000 items/sec.
Your `consumer` is slow. It writes to Postgres at 50 items/sec.
**Result:** The "Lag" (Backlog) grows infinitely. You will never catch up.

## 2. Vertical Scaling (Bad)
"Make the computer faster."
This has limits. You can't upgrade CPU forever.

## 3. Horizontal Scaling (Good)
"Run 10 Consumers."
Kafka has a magic feature called **Consumer Groups**.

If you start `consumer.py` **TWICE** (in two terminals) with the SAME `group_id='db_writer_group'`:
1.  Kafka detects the new worker.
2.  Kafka splits the data.
    *   Worker 1 gets items 1, 3, 5...
    *   Worker 2 gets items 2, 4, 6...
3.  **Result:** You process 100 items/sec (Doubled speed).

## 4. Partitions
For this to work, your Kafka Topic must have **Multiples Partitions**.
*   If Topic has 1 Partition -> Only 1 Consumer can read (Others sit idle).
*   If Topic has 10 Partitions -> You can have up to 10 Consumers reading in parallel.

When we started Redpanda (Day 30), it usually creates 1 partition by default.
To scale, you might need to create the topic manually with partitions:
```bash
docker exec -it redpanda rpk topic create scraped_items -p 5
```

## 5. Homework
1.  Open 3 terminals.
2.  Run `python consumer.py` in ALL 3.
3.  Run `producer.py`.
4.  Watch them print messages. You will see that they "share" the load.
    *   Terminal A: gets some.
    *   Terminal B: gets some.
    *   Terminal C: gets some.

## 6. Graduation
You have completed Week 5.
*   **Week 1:** Wire.
*   **Week 2:** Browser.
*   **Week 3:** API.
*   **Week 4:** Mobile.
*   **Week 5:** Infrastructure (Kafka).

**Next Week:** Orchestration.
Running a script is easy. Running 100 scripts on a schedule with error handling and retries requires **Airflow**.
