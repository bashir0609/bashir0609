# Day 53: The Dashboard (Streamlit)

**Objective:** See the data.
**Tool:** Streamlit (Python).

---

## 1. Why Streamlit?
It turns Python scripts into Web Apps.
Perfect for "Internal Tools" or "Portfolios".

## 2. The Code (`dashboard.py`)

```python
import streamlit as st
import pandas as pd
import psycopg2
import os

# 1. Connect to DB
conn = psycopg2.connect(
    host=os.getenv("DB_HOST", "localhost"),
    database="flights",
    user="user",
    password="password"
)

# 2. Fetch Data
@st.cache_data
def load_data():
    query = "SELECT * FROM flights ORDER BY dept_time DESC"
    return pd.read_sql(query, conn)

df = load_data()

# 3. Layout
st.title("✈️ Flight Tracker")
st.write(f"Tracking {len(df)} flights.")

# 4. Filters
airline = st.selectbox("Airline", df['airline'].unique())
filtered_df = df[df['airline'] == airline]

# 5. Chart
st.line_chart(filtered_df.set_index('dept_time')['price'])

# 6. Table
st.dataframe(filtered_df)
```

## 3. Integration
Add this to `docker-compose.yml`:

```yaml
  dashboard:
    image: python:3.9
    command: sh -c "pip install streamlit psycopg2-binary pandas && streamlit run dashboard.py"
    volumes:
      - ./dashboard.py:/dashboard.py
    ports: ["8501:8501"]
    depends_on:
      - postgres
```

## 4. Homework
1.  Add the Dashboard service.
2.  Run `docker-compose up`.
3.  Go to `localhost:8501`.
4.  See your scraped data in a beautiful localized chart.
