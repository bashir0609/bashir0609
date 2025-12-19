# Day 38: The Logic (Advanced DAGs)

**Objective:** "If Scraper A fails, try Scraper B."
**Concept:** `TriggerRule` & Branching.

---

## 1. The Fallback Pattern
In scraping, failure is normal.
If `Task A (Standard Request)` fails, we want to trigger `Task B (Selenium Request)`.

By default, Airflow stops if a task fails. We change this with `trigger_rule`.

```python
from airflow.utils.trigger_rule import TriggerRule

task_a = PythonOperator(task_id='standard_scrape', ...)

task_b = PythonOperator(
    task_id='selenium_scrape',
    trigger_rule=TriggerRule.ALL_FAILED, # Only run if A fails
    ...
)

task_a >> task_b
```

## 2. The Branching Pattern
Maybe you only want to scrape on Weekdays.
Use `BranchPythonOperator`.

```python
from airflow.operators.python import BranchPythonOperator

def check_day():
    if datetime.now().weekday() < 5:
        return 'scrape_task_id'
    else:
        return 'skip_task_id'

branch = BranchPythonOperator(
    task_id='check_weekday',
    python_callable=check_day
)

scrape = PythonOperator(...)
skip = DummyOperator(task_id='skip_task_id')

branch >> [scrape, skip]
```

## 3. The Retry Policy
Never write `try/except` loops inside your scraper for retries. Let Airflow handle it.

```python
default_args = {
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}
```
If the scraper crashes (Exception), Airflow waits 5 mins and tries again. After 3 fails, it marks it "FAILED" and sends an alert (Day 40).

## 4. Homework
1.  Modify your DAG from Day 37.
2.  Make the scraper function `raise Exception("Boom")`.
3.  Add a second task that runs ONLY if the first one fails (Fallback).
4.  Watch it in action in the UI graph view.
