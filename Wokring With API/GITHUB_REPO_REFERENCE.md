# Agentic AI Roadmap — API learning assets (GitHub)

## Repository

- **Remote URL:** `https://github.com/Ch-Balaji/AgenticAI_Roadmap_Learning_Assets.git`
- **Owner:** `Ch-Balaji`
- **Repo name:** `AgenticAI_Roadmap_Learning_Assets`

## What’s in this folder (`Wokring With API/` — matches clone layout)

| File | Description |
|------|-------------|
| `foundations_apis_and_requests.md` | Session notes: APIs, HTTP, JSON, `requests`, API keys |
| `calling_openai_in_practice.ipynb` | Hands-on: OpenAI + Groq-style calls |
| `.env.example` | Template — copy to `.env` locally (same folder) |
| `README.md` | Quick start |

At the repository root there is only a stub `README.md` and `.gitignore` (ignored patterns apply across the repo).

## Local setup

1. Clone the repo (or pull latest).
2. `cp .env.example .env`
3. Add your `OPENAI_API_KEY` and `GROQ_API_KEY` to `.env`.
4. Run the notebook with the interpreter that has `openai`, `requests`, `python-dotenv`, etc.

## Authentication (tokens)

**Do not commit** GitHub personal access tokens or API keys into this repo.

- Prefer **SSH** (`git@github.com:Ch-Balaji/AgenticAI_Roadmap_Learning_Assets.git`) or GitHub CLI (`gh auth login`).
- For HTTPS with a PAT, use the macOS Keychain credential helper or a short-lived token in your terminal session only—not in tracked files.

---

### Security note if a token was ever pasted in chat or committed

Immediately **revoke** that token on GitHub and generate a **new** one.
