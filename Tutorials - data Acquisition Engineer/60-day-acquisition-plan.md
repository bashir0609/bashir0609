# 📅 60-Day Data Acquisition Master Plan

**Goal:** Go from "Senior Scraper" to "Data Acquisition Engineer" in 8 weeks.
**Commitment:** ~1-2 hours per day.

---

## 🟢 Weeks 1-2: The "Stealth" Layer (Anti-Bot & Fingerprinting)
*Goal: Understand why you get blocked and how to spoof TLS.*

### Week 1: TLS & Fingerprinting
*   **Day 1:** Read about "JA3 Fingerprinting". Understand how Cloudflare identifies Python requests vs Chrome requests.
*   **Day 2:** Install `tshark` (Wireshark CLI). Inspect the "Client Hello" packet of a `requests.get()` vs a Browser request. See the difference.
*   **Day 3:** Implementation: Write a script using **`curl_cffi`** or **`tls_client`** in Python. Make a request to `tls.peet.ws` and ensure it mimics a Real Chrome Browser.
*   **Day 4:** Deep Dive: HTTP/2 vs HTTP/1.1. Force your scraper to use HTTP/2.
*   **Day 5:** Mini-Project: Scrape a Cloudflare-protected site (e.g., G2 or Glassdoor) *without* using Selenium/Playwright—only requests with TLS spoofing.
*   **Day 6-7:** Rest / Buffer.

### Week 2: Browser Patching (When you MUST use a browser)
*   **Day 8:** Learn about `navigator.webdriver` flags and `cdc_` variables.
*   **Day 9:** Set up **Nodriver** (Undetected-Chromedriver successor). Compare it against standard Selenium.
*   **Day 10:** Learn **Canvas Fingerprinting**. Use a library to inject random noise into the canvas readout.
*   **Day 11:** **AudioContext Fingerprinting**. How to spoof it.
*   **Day 12:** Build a "Stealth Driver" class that passes 100% on `pixelscan.net` and `creepjs`.
*   **Day 13-14:** Rest / Buffer.

---

## 🟡 Weeks 3-4: The "API & Mobile" Layer (Reverse Engineering)
*Goal: Stop parsing HTML. Find the hidden JSON.*

### Week 3: Web API Reverse Engineering
*   **Day 15:** Master Chrome DevTools > Network Tab. Learn "Copy as Curl". Learn to replay requests in Postman.
*   **Day 16:** Handling "Signed" Requests. Find the JS function that generates the `X-Signature` header.
*   **Day 17:** Intro to **AST (Abstract Syntax Tree)** parsing. How to read minified JavaScript to find API secrets.
*   **Day 18:** Project: Reverse Engineer a complex site (e.g., Twitter/X guest token flow or Instagram).
*   **Day 19-21:** Practice on 3 different sites. Documentation.

### Week 4: Mobile App Interception (MITM)
*   **Day 22:** Install **MITMProxy** on your PC.
*   **Day 23:** Configure an Android Emulator (Genymotion or Android Studio) to route traffic through MITMProxy.
*   **Day 24:** SSL Pinning. What is it? How to bypass it using **Frida** or **Objection**.
*   **Day 25:** Intercept traffic from a simple app (e.g., a News app).
*   **Day 26:** Intercept traffic from a hard app (e.g., a Food Delivery app).
*   **Day 27-28:** Rest.

---

## 🔴 Weeks 5-6: The "Infrastructure" Layer (Scale & Reliability)
*Goal: Handle 100GB/day. Decouple scraping from storage.*

### Week 5: Streaming & Buffers
*   **Day 29:** Why DBs fail. Intro to **Kafka** (or Redpanda).
*   **Day 30:** Run Kafka in Docker. Write a Python "Producer" that sends dummy scraped data.
*   **Day 31:** Write a Python "Consumer" that reads from Kafka and saves to Postgres.
*   **Day 32:** Implement **Schema Registry**. Ensure if the scraper sends bad data, the Consumer rejects it (Dead Letter Queue).
*   **Day 33:** Scaling: Run 5 Consumers in parallel to drain the queue faster.
*   **Day 34-35:** Rest.

### Week 6: Orchestration (Airflow)
*   **Day 36:** Setup **Apache Airflow** locally.
*   **Day 37:** Write a DAG that runs a scraper every hour.
*   **Day 38:** Advanced DAGs: "Trigger Rule". If Scraper A fails, run Scraper B (Backup).
*   **Day 39:** Dynamic DAGs. Generate 100 DAGs for 100 different news sites automatically.
*   **Day 40:** Monitoring. Set up a Slack Alert if 0 rows are scraped.
*   **Day 41-42:** Rest.

---

## ⚫ Weeks 7-8: The Capstone Project (Proof of Work)
*Goal: Build the "Airline Price Tracker" to show on your CV.*

### Week 7: Building the Engine
*   **Day 43-45:** Reverse Engineer the API of a major flight site (e.g., Skyscanner). Bypass their Akamai/Bot protection using your TLS skills.
*   **Day 46-47:** Build the Scraper. It must be headless (no browser). It must push data to Kafka.
*   **Day 48-49:** Build the Consumer. Validate data with Pydantic. Save to Postgres.

### Week 8: Polish & Deploy
*   **Day 50-52:** Containerize everything (Scraper, Kafka, DB, Airflow) in `docker-compose.yml`.
*   **Day 53-54:** Add a simple Dashboard (Streamlit or Vue) to show "Cheapest Flight to London".
*   **Day 55:** Write the `README.md`. Explain your "Anti-Bot Strategy" and "Architecture".
*   **Day 56-59:** Deploy to a cheap VPS (Hetzner ($5/mo) or AWS Free Tier).
*   **Day 60:** **PROJECT LAUNCH.** Add to LinkedIn. Apply for jobs.
