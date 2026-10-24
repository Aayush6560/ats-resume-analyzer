# ATS Resume Analyzer

An AI-powered resume analyzer with a FastAPI backend and Streamlit frontend.
It scores resumes against ATS criteria, compares them with a job description,
and provides actionable feedback.

## Setup

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m spacy download en_core_web_sm
Copy-Item .env.example .env
```

Fill in the Supabase and Groq values in `.env` before using authentication or
the LLM-powered parsing features.

## Run the backend

```powershell
python -m uvicorn backend.main:app --reload --port 8000
```

The API is available at `http://localhost:8000`, with interactive
documentation at `/docs`.

## Run the frontend

In a second terminal:

```powershell
streamlit run frontend/streamlit_app.py
```
