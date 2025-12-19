# Role Spotlight: Data Acquisition Engineer

This is the **"Corporate Version"** of a Web Scraping Expert.
It is stable, well-paid ($90k - $160k), and respects the complexity of the work.

---

## 🧐 What is it?
A Data Acquisition Engineer doesn't just "write scripts." They build **infrastructure** to ingest external data reliably, legally, and at scale.
Companies that hire these: **Hedge Funds, AI Startups, Real Estate Tech, Market Research Firms, Travel Aggregators.**

## 📋 Key Responsibilities (vs. Freelance Scraper)

| Feature | Freelance Scraper | Data Acquisition Engineer |
| :--- | :--- | :--- |
| **Goal** | "Get this file by Friday." | "Maintain a live feed of competitor pricing forever." |
| **Scale** | 10k - 100k rows. | 10M+ rows per day. |
| **Quality** | "It looks mostly right." | **Schema Validation (Pydantic)**. Data *must* be perfect to enter the warehouse. |
| **Compliance** | "Use any proxy." | **Robots.txt compliance, Terms of Service reviews, GDPR compliance.** |
| **Stack** | Python script on local PC. | **Airflow + Docker + Kubernetes + AWS Lambda + Kafka.** |

---

## 🛠️ The Tech Stack You Need to Show
To get this job, your resume needs to show you think about the *system*, not just the *site*.

1.  **Orchestration:** "I use **Airflow** or **Prefect** to schedule 500 concurrent spiders."
2.  **Validation:** "I use **Pydantic** to validate every row. If the schema changes (e.g., price becomes null), the pipeline alerts me via Slack."
3.  **Storage:** "I don't just save CSVs. I stream data into **Kafka** or load it into **Snowflake** / **S3 Data Lakes**."
4.  **Monitoring:** "I use **Prometheus/Grafana** to track proxy failure rates and success rates per domain."

---

## 💎 Resume Bullet Points (Copy-Paste)
*   *Designed and maintained a distributed data acquisition pipeline ingesting 5M+ records daily from 20+ sources using **Python**, **Scrapy**, and **AWS Lambda**.*
*   *Reduced proxy costs by 40% by implementing intelligent rotation logic and caching strategies using **Redis**.*
*   *Built a robust anti-bot system capable of bypassing Cloudflare/Akamai using customized **Playwright** patches and TLS fingerprinting.*
*   *Implemented automated data quality checks with **Great Expectations**, reducing downstream ETL errors by 90%.*

---

## 🧠 Interview Questions to Expect
1.  *"How do you handle a site that changes its HTML layout every week?"* (Answer: Structural vs Visual selectors, monitoring, ML-based extraction).
2.  *"We need to scrape 1M pages in an hour. How do you architect this?"* (Answer: Distributed workers, async I/O, queue systems).
3.  *"How do you ensure we don't get sued?"* (Answer: Respecting crawl rates, identifying as a bot agent where possible, only scraping public data).

---

## 🚀 Verdict
This is the **perfect** title for you. It leverages your "Systems Architect" mindset.
Stop calling yourself a "Scraper" on LinkedIn. Change your headline to:
**Data Acquisition Engineer | High-Scale Web Scraping & Python Automation**
