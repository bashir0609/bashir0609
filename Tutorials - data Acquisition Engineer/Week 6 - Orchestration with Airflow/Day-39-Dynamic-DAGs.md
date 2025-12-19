# Day 39: The Factory (Dynamic DAGs)

**Objective:** Create 100 Scrapers for 100 different news sites without writing 100 files.
**Concept:** Dynamic DAG Generation.

---

## 1. The Variable
Airflow parses Python. This means we can use `for` loops.

Imagine you have a configuration (JSON/DB):
```python
targets = [
    {"id": "bbc", "url": "bbc.co.uk"},
    {"id": "cnn", "url": "cnn.com"},
    # ... 100 more
]
```

## 2. The Factory Pattern
Save this as `dags/dynamic_news_factory.py`.

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

# 1. Configuration
targets = ["bbc", "cnn", "fox", "nbc"]

def create_dag(site_id):
    
    def scrape_func():
        print(f"Scraping {site_id}...")
    
    dag = DAG(
        dag_id=f'scraper_{site_id}', # Unique ID
        default_args={'owner': 'islah', 'start_date': datetime(2024, 1, 1)},
        schedule_interval='@daily',
        catchup=False
    )
    
    with dag:
        t1 = PythonOperator(
            task_id='scrape',
            python_callable=scrape_func
        )
        
    return dag

# 2. Key Step: Register DAGs in global namespace
for site in targets:
    dag_object = create_dag(site)
    # Airflow looks for objects in globals()
    globals()[f'scraper_{site}'] = dag_object
```

## 3. The Result
1.  Refresh Airflow UI.
2.  You will see 4 generic DAGs:
    *   `scraper_bbc`
    *   `scraper_cnn`
    *   ...
3.  Each runs independently. If CNN fails, BBC continues.

## 4. Why this is "Senior"
Junior developers copy-paste `scraper_bbc.py` to `scraper_cnn.py`.
Senior developers write **One Engine** and feed it configurations.
When you need to update the logic, you update ONE file, and all 100 scrapers are updated.

## 5. Homework
1.  Use the code above.
2.  Refresh Airflow.
3.  Verify you see multiple DAGs generated from the list.
