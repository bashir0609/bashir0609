# Day 37: The Script (First DAG)

**Objective:** Write a scraper that runs automatically every morning.
**Concept:** `PythonOperator` & CRON Expressions.

---

## 1. Structure of a DAG
A DAG file is just Python.
It needs:
1.  **Arguments:** (Owner, Start Date, Retries).
2.  **DAG Definition:** (Schedule, ID).
3.  **Operators:** (The actual tasks).

## 2. The Code

Save this as `dags/daily_scraper.py`.

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime
import requests

# 1. The Scraping Function
def scrape_hn():
    print("🚀 Starting Scraper...")
    resp = requests.get("https://hacker-news.firebaseio.com/v0/topstories.json")
    data = resp.json()[:5]
    print(f"✅ Found {len(data)} items: {data}")
    # In real life, you push this to Kafka/DB here

# 2. Defaults
default_args = {
    'owner': 'islah',
    'start_date': datetime(2024, 1, 1),
    'retries': 1,
}

# 3. The DAG
with DAG(
    'daily_hackernews_v1',
    default_args=default_args,
    schedule_interval='0 8 * * *', # At 08:00 AM every day
    catchup=False # Don't run for past dates
) as dag:
    
    # 4. The Task
    task1 = PythonOperator(
        task_id='scrape_task',
        python_callable=scrape_hn
    )

    t1
```

## 3. Deployment
1.  Save the file.
2.  Refresh the Airflow UI (it parses files every 30s).
3.  You should see `daily_hackernews_v1`.
4.  Toggle the **Unpause** switch (Blue switch).
5.  Click the "Play" button to trigger it manually.
6.  Click the task -> **Logs** to see your "Found X items" print statement.

## 4. Homework
1.  Create the DAG.
2.  Trigger it.
3.  Find the logs.
4.  Change the schedule to run every minute (`* * * * *`). Watch it go crazy.
