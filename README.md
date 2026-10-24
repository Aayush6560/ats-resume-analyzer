# ATS Resume Analyzer

An AI-powered resume analysis platform that helps candidates improve their
resumes for Applicant Tracking Systems (ATS). The project combines a FastAPI
backend for document processing and analysis with a Streamlit frontend for
authentication, scoring, feedback, history, and PDF reports.

## Features

- Upload and parse PDF, DOC, and DOCX resumes
- Calculate an overall ATS score from:
  - Formatting
  - Keyword relevance
  - Resume content
  - Skill validation
  - ATS compatibility
- Compare a resume with an optional job description
- Identify matched keywords, missing keywords, and skill gaps
- Validate whether listed skills are supported by project or experience content
- Generate detailed feedback with explanations and improvement examples
- Save and review analysis history through Supabase
- Export analysis results as a PDF report
- Email/password and Google OAuth authentication through Supabase
- Interactive FastAPI documentation through Swagger UI and ReDoc

## Architecture

```text
Streamlit frontend
        |
        | HTTP requests + Supabase authentication
        v
FastAPI backend
        |
        +-- Resume parsing (PDF/DOC/DOCX)
        +-- spaCy NLP analysis
        +-- Sentence Transformer semantic matching
        +-- Optional Groq-powered parsing
        +-- Supabase history storage
        +-- HTML/PDF report generation
```

### Main directories

| Directory | Purpose |
| --- | --- |
| `backend/api` | API routes and JWT authentication |
| `backend/core` | Application and environment configuration |
| `backend/services` | Resume parsing, scoring, matching, feedback, and reports |
| `backend/templates` | HTML templates used for PDF reports |
| `frontend/components` | Reusable Streamlit UI components |
| `frontend/services` | Frontend API and Supabase clients |
| `frontend/views` | Streamlit application pages |
| `jupyter notebooks` | Exploratory data and model notebooks |

## Requirements

- Python 3.10 or newer
- A Supabase project for authentication and analysis history
- A Groq API key for features that use the LLM parser
- Internet access on first run so the Sentence Transformer model can be
  downloaded
- On Linux, WeasyPrint may require system packages for Cairo and Pango

## Installation

From the repository root:

### Windows PowerShell

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m spacy download en_core_web_sm
Copy-Item .env.example .env
```

### macOS/Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m spacy download en_core_web_sm
cp .env.example .env
```

The application prefers `en_core_web_md` for improved accuracy and falls back
to `en_core_web_sm` when the larger model is not installed. To use the larger
model, install it separately:

```bash
python -m spacy download en_core_web_md
```

## Configuration

Copy `.env.example` to `.env` and set the values for your environment:

| Variable | Required | Description |
| --- | --- | --- |
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_KEY` | Yes for history writes | Supabase service-role key used only by the backend |
| `SUPABASE_ANON_KEY` | Yes for frontend auth | Public Supabase anonymous key |
| `SUPABASE_JWT_SECRET` | Conditional | JWT secret for HS256 token verification |
| `GROQ_API_KEY` | Optional | Enables Groq-powered resume parsing |
| `SENTENCE_TRANSFORMER_MODEL` | Optional | Embedding model; defaults to `all-MiniLM-L6-v2` |
| `AUTH_REDIRECT_URL` | Optional | OAuth callback URL; defaults to `http://localhost:8501` |

Never commit `.env`, Supabase service-role keys, JWT secrets, or API keys.
Local Streamlit secrets are also excluded by `.gitignore`.

## Running the application

Start the backend from the repository root:

```bash
python -m uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

The backend will load the NLP and embedding models during startup. In a
separate terminal, activate the same virtual environment and start the
frontend:

```bash
streamlit run frontend/streamlit_app.py
```

Open the Streamlit URL shown in the terminal, normally
`http://localhost:8501`.

The frontend uses `http://localhost:8000` by default for the backend. For a
deployed frontend, configure the backend URL in Streamlit secrets:

```toml
[backend]
url = "https://your-backend.example.com"
```

## API endpoints

The API is served under `/api/v1`:

| Method | Endpoint | Authentication | Description |
| --- | --- | --- | --- |
| `GET` | `/api/v1/health` | No | Check that the API and models are ready |
| `POST` | `/api/v1/analyze-resume` | Yes | Analyze a PDF, DOC, or DOCX resume |
| `GET` | `/api/v1/history` | Yes | Get the signed-in user's analyses |
| `DELETE` | `/api/v1/history/{analysis_id}` | Yes | Delete a saved analysis |
| `POST` | `/api/v1/generate-pdf` | Yes | Generate a PDF from analysis data |
| `GET` | `/api/v1/history/{analysis_id}/pdf` | Yes | Generate a PDF for saved history |

Interactive documentation is available at:

- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

## Resume upload limits

- Supported formats: PDF, DOC, and DOCX
- Maximum file size: 5 MB
- Job descriptions are optional, but providing one enables keyword and semantic
  comparison results

## Verification

Run the syntax and dependency checks from the repository root:

```bash
python -m compileall -q backend frontend
python -m pip check
```

To verify that the API starts and both models load, call:

```bash
curl http://localhost:8000/api/v1/health
```

Expected response:

```json
{
  "status": "healthy",
  "nlp_loaded": true,
  "embedder_loaded": true
}
```

## Troubleshooting

### `en_core_web_sm` or `en_core_web_md` is not found

Install the fallback model:

```bash
python -m spacy download en_core_web_sm
```

### Authentication is not configured

Set `SUPABASE_URL` and `SUPABASE_ANON_KEY` for the frontend. The backend also
needs either `SUPABASE_JWT_SECRET` for HS256 tokens or a reachable Supabase URL
for JWKS-based verification.

### History is unavailable

Confirm that `SUPABASE_URL`, `SUPABASE_KEY`, and the required Supabase table and
permissions are configured. History save failures do not prevent an analysis
response from being returned.

### PDF generation fails on Linux

Install the system libraries required by WeasyPrint. For Debian/Ubuntu:

```bash
sudo apt install -y libcairo2 libpango-1.0-0 libpangoft2-1.0-0 libffi-dev
```

## License

No license has been specified yet. Add a license before distributing or
reusing this project publicly.
