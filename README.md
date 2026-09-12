# Investor Data Chat Assistant

A two-tab Streamlit app that pairs a fixed-view analytics dashboard with a natural-language chat interface over the same underlying data. Built for an investor-relations team so non-technical staff could ask ad-hoc questions instead of waiting on a new dashboard view every time.

## What it does

- **Dashboard tab** — KPIs, monthly investment trend, order-stage breakdown, top-investor list, investor segmentation (tier × activity status), and a filterable order table. Computed straight from SQL, no LLM involved, so it keeps working even if the AI side is down.
- **Ask a Question tab** — a Gemini-powered chat that answers free-form questions ("How many investors are in the VIP tier?", "Show me active orders for Project X") by calling the exact same query functions as tools. It isn't retrieving from documents or generating numbers itself — every answer is grounded in a live function call to the database, so it can't drift from what the dashboard shows.
- Session-scoped passkey gate, retry handling and friendly error messages around the Gemini API, and periodic conversation resets to bound token cost in long chat sessions.

## Architecture

```
Google Sheets (source data)
        │
   scripts/  (sync / ETL)
        ▼
   PostgreSQL
        │
  src/queries.py   ← single source of truth for all data access
   ├────────────┐
   ▼            ▼
src/llm_tools.py   app.py (Dashboard tab)
   │
src/chatbot.py  (Gemini + function calling)
```

Routing both the dashboard and the chatbot through one query layer means they can never disagree with each other, and the dashboard doesn't depend on the Gemini API being available.

## A couple of the trickier bits

- Function-calling tools are deliberately lightweight — e.g. a dedicated "count" tool instead of returning full row data — to keep token usage down on the kind of questions ("how many...") that come up constantly.
- Conversation history is capped and the session resets after a set number of turns, so a long chat doesn't quietly balloon the context sent to the model on every turn.

## Stack

Python · Streamlit · PostgreSQL (SQLAlchemy) · Google Sheets API (gspread) · Google Gemini API (function calling)

## Run locally

```bash
pip install -r requirements.txt
cp .env.example .env   # fill in DB, Gemini, and Sheets credentials
streamlit run app.py
```

## Note

Built during an AI/prompt-engineering internship for internal investor-relations use at an agritech startup. This is the general application code — no company data, spreadsheets, credentials, or investor information are included.
