# RepoMind

> Understand any GitHub repository without digging through the entire codebase.

RepoMind takes the URL of a public GitHub repository and returns a structured explanation of it: what it does, how it is organised, its main components, technologies, entry points, and how data flows between the parts.

**Paste a repository → Analyze it → Understand it.**

---

## Features

- **One URL in, one explanation out.** Paste any public GitHub repository link.
- **Overview and architecture:** a short summary of what the project does and how it is laid out.
- **Components and technologies:** the main parts of the codebase and the stack it uses.
- **Entry points:** where the program starts, so you know which file to open first.
- **Relationships and data flow:** how the components talk to each other.
- **Grounded answers:** the model is told to use only what the repository supports and not to invent details.
- **Copy in one click:** copy the generated analysis straight from the page.
- **Neo-Brutalist frontend** that works on desktop and mobile.

---

## How it works

```text
GitHub repository URL
        │
        ▼
FastAPI backend  (POST /analyze)
        │
        ├── Parse the URL into owner / repository
        ├── Fetch metadata and the file tree (GitHub REST API)
        ├── Filter the tree: skip build folders, dependencies, binaries,
        │   media and files over 500 KB; keep key files such as README,
        │   package.json, requirements.txt, pyproject.toml
        ├── Build a compact representation of the repository
        ├── Summarise it with Google Gemini (structured JSON output)
        └── Work out entry points, components, relationships and data flow
        │
        ▼
Frontend  →  readable analysis
```

---

## Tech stack

| Part | Built with |
| --- | --- |
| Backend | Python, FastAPI, Uvicorn, Pydantic, httpx |
| AI | Google Gemini via `google-genai` (default model `gemini-2.5-flash-lite`) |
| Data source | GitHub REST API |
| Frontend | HTML, CSS, vanilla JavaScript |

---

## Project structure

```text
RepoMind/
├── Backend/
│   ├── app/
│   │   ├── main.py                     # FastAPI app, CORS, /health
│   │   ├── api/routes.py               # POST /analyze
│   │   ├── core/config.py              # Environment variables
│   │   ├── models/                     # Request and response models
│   │   └── services/
│   │       ├── github_service.py       # GitHub API calls
│   │       ├── file_filter.py          # Picks the files worth reading
│   │       ├── repository_service.py   # Builds the repository representation
│   │       ├── language_service.py     # Detects file languages
│   │       ├── gemini_service.py       # AI summary
│   │       ├── analysis_service.py     # Puts the analysis together
│   │       └── architecture_service.py # Entry points, relationships, data flow
│   └── requirements.txt
├── Frontend/
│   ├── index.html
│   ├── styles.css
│   └── app.js
└── LICENSE
```

---

## Getting started

### 1. Clone

```bash
git clone https://github.com/AgrimJoshi1/RepoMind.git
cd RepoMind/Backend
```

### 2. Install

```bash
python -m venv venv
# Windows: venv\Scripts\activate    macOS/Linux: source venv/bin/activate
pip install -r requirements.txt
```

### 3. Add your keys

Create `Backend/.env`:

```env
GEMINI_API_KEY=your_gemini_api_key
# Optional
GEMINI_MODEL=gemini-2.5-flash-lite
GITHUB_TOKEN=your_github_token
```

- `GEMINI_API_KEY` is **required**; the server will not start without it. Get one from [Google AI Studio](https://aistudio.google.com/).
- `GITHUB_TOKEN` is optional but recommended: without it, GitHub's unauthenticated rate limit is low.

### 4. Run the backend

```bash
uvicorn app.main:app --reload
```

The API runs at `http://localhost:8000`. Check it at `http://localhost:8000/health`.

### 5. Run the frontend

The backend only accepts requests from port **5500**, so serve the frontend there:

```bash
cd ../Frontend
python -m http.server 5500
```

Or open `Frontend/index.html` with VS Code's **Live Server**, which uses port 5500 by default.

Open `http://localhost:5500`, paste a public GitHub URL and click **Analyze**.

---

## API

### `POST /analyze`

```json
{ "github_url": "https://github.com/owner/repository" }
```

Returns:

```json
{
  "repository": { "name": "", "full_name": "", "description": "", "default_branch": "", "language": "" },
  "summary": {
    "overview": "",
    "architecture": "",
    "main_components": [],
    "technologies": [],
    "entry_points": []
  },
  "architecture": {
    "entry_points": [],
    "components": [],
    "relationships": [],
    "data_flow": []
  }
}
```

### `GET /health`

Returns `{ "status": "Working", "service": "RepoMind" }`.

---

## Limitations

- Public repositories only.
- Large repositories are reduced to the most relevant files, and files over 500 KB are skipped.
- The analysis is AI-generated; treat it as a starting map of the codebase, not a full audit.

---

## Built by

- **Agrim Joshi** · [GitHub](https://github.com/AgrimJoshi1)
- **Ayush Sharma** · [GitHub](https://github.com/thisIsAyushFr)

---

## License

[MIT](LICENSE)
