# Day 43: The Recon (Capstone - Analysis)

**Objective:** Map out the battlefield. Find the API endpoints.
**Target:** A Flight Search Engine (General Example).

---

## 1. The Challenge
Flight sites are complex.
1.  **Session:** You search for "LHR to JFK".
2.  **Job:** The server starts a background job to query airlines.
3.  **Polling:** The frontend hits `/api/poll` every 2 seconds to get updates.

**If you just hit the Search URL, you get an empty list.**
You must reverse engineer the **Polling Flow**.

## 2. DevTools Strategy
1.  Open Network Tab.
2.  Search for flights.
3.  Watch the waterfall.
    *   **POST /create-session:** Sends `{ from: "LHR", to: "JFK", date: "..." }`.
    *   **Response:** `{ session_id: "abc-123" }`.
    *   **GET /poll/abc-123:** Returns `{ status: "IN_PROGRESS", flights: [] }`.
    *   **GET /poll/abc-123:** Returns `{ status: "COMPLETE", flights: [...] }`.

## 3. The Headers
The endpoints will likely be protected by:
*   `X-Client-Version`
*   `Device-Id`
*   `Cookie`: Heavy session cookies.

**Task:** Use "Copy as cURL" on the **Poll** request. Verify if it works in Postman without cookies (it probably won't). You need to find which request *sets* the cookie (usually the first POST).

## 4. Homework
1.  Pick a site (Skyscanner/Kayak/Momondo).
2.  Map the flow on paper:
    *   Step 1: Get Session.
    *   Step 2: Poll Results.
3.  Verify you can trigger it manually in Postman.
