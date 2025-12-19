# Day 36: The Conductor (Airflow Setup)

**Objective:** Stop using `crontab` and `while True`. Use a real Scheduler.
**Tool:** Apache Airflow.

---

## 1. Why not cron?
*   **Cron:** Runs script at 10:00 AM. If it fails? No retries. If the logs are lost? No idea.
*   **Airflow:** Runs script at 10:00 AM.
    *   Fails? Retries 3 times.
    *   Still fails? Sends Slack Alert.
    *   Success? Triggers the Next Task (e.g., "Upload to S3").
    *   **Visibility:** A Web UI showing exactly what happened.

## 2. Installation (Docker)
Airflow is complex (Webserver, Scheduler, Database, Workers). You MUST use Docker.

1.  **Download docker-compose:**
    ```bash
    curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.8.1/docker-compose.yaml'
    ```
2.  **Initialize DB:**
    ```bash
    docker-compose up airflow-init
    ```
3.  **Start:**
    ```bash
    docker-compose up -d
    ```

## 3. The UI
1.  Go to `localhost:8080`.
2.  User/Pass: `airflow` / `airflow`.
3.  You will see example DAGs.
4.  **DAG** = Directed Acyclic Graph (A workflow).

## 4. Where do scripts go?
The `docker-compose.yaml` usually mounts a local folder `./dags` to `/opt/airflow/dags`.
Any Python file you put in `./dags` will appear in the UI.

## 5. Homework
1.  Get Airflow running locally.
2.  Login to the UI.
3.  Delete the example DAGs (Optional: Set `AIRFLOW__CORE__LOAD_EXAMPLES=False` in docker-compose).
