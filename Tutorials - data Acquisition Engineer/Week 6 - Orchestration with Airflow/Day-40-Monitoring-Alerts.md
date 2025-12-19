# Day 40: The Watchtower (Monitoring & Alerts)

**Objective:** Get a Slack/Discord notification when a scraper fails.
**Concept:** `on_failure_callback`.

---

## 1. The Callback
Airflow allows you to define a function that runs **only if a task fails**.

```python
def slack_alert(context):
    exception = context.get('exception')
    task_id = context.get('task_instance').task_id
    
    msg = f"🚨 ALERT: Task {task_id} failed!\nError: {exception}"
    
    # Send to Slack/Discord Webhook
    requests.post("https://hooks.slack.com/...", json={"text": msg})
```

## 2. Attaching it to the DAG

```python
default_args = {
    'on_failure_callback': slack_alert, # <--- Magic happens here
    'email_on_failure': False,
    'retries': 1
}

with DAG(..., default_args=default_args) as dag:
    # ...
```

## 3. SLA (Service Level Agreements)
You can also set deadlines.
"If this task takes longer than 1 hour, alert me."

```python
task = PythonOperator(
    ...,
    sla=timedelta(hours=1)
)
```

## 4. Graduation
You have completed Week 6.
*   **Week 1:** Wire.
*   **Week 2:** Browser.
*   **Week 3:** API.
*   **Week 4:** Mobile.
*   **Week 5:** Kafka.
*   **Week 6:** Airflow.

**Next Week:** The Capstone Project.
We will build the **Airline Price Tracker**.
*   Reverse Engineer Skyscanner (API).
*   Stream to Kafka (Buffer).
*   Save to Postgres (Consumer).
*   Orchestrate with Airflow (Scheduler).
*   Monitor with Slack (Alerts).
