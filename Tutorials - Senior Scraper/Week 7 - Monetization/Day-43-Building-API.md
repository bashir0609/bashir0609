# Day 43: The Wrapper (FastAPI)

**Objective:** Turn your script `scrape.py` into `http://localhost:8000/scrape?url=...`
**Why?** You can't sell a `.py` file easily. You CAN sell an API endpoint.

---

## 1. The Concept
Clients don't want to run Python. They want to send a URL and get JSON back.
**FastAPI** is the modern standard for this. It's fast, async (likes scraping), and auto-documents itself.

## 2. The Setup
```bash
pip install fastapi uvicorn
```

## 3. The Code
Create `main.py`:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
# Import your scraper logic from previous weeks
# from my_scraper import scrape_google_maps 

app = FastAPI()

class ScrapeRequest(BaseModel):
    url: str
    premium: bool = False

@app.get("/")
def home():
    return {"status": "Online", "message": "Scraper API v1"}

@app.post("/scrape")
def trigger_scrape(request: ScrapeRequest):
    print(f"Received request for {request.url}")
    
    try:
        # call your scraping function here
        # data = scrape_google_maps(request.url)
        
        # Mock data for now
        data = {"title": "Example Business", "phone": "555-0199"}
        
        return {
            "status": "success",
            "data": data
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## 4. Running it
1.  Run `python main.py`.
2.  Open `http://localhost:8000/docs`.
3.  You see an interactive Swagger UI.
4.  Click **POST /scrape**, enter a URL, and click **Execute**.
5.  You just built a SaaS.

## 5. Homework
1.  Take your **Google Maps** script from Week 5.
2.  Wrap it in FastAPI.
3.  Make it so you can query `http://localhost:8000/maps?q=Pizza` and get the JSON list back.
