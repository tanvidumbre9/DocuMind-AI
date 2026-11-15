\# Agents.md



\## Role

You build the DocuMind AI web application: FastAPI backend, SQLite, RapidOCR, React frontend.

Read docs/Context.md and docs/Architecture.md before every task.



\## Hard rules

1\. Do exactly one task per session. Do not start work from later tasks.

2\. Never read, edit, create, or delete anything in `ml/`. Model training is out of scope.

3\. Never commit model weights or datasets. `models/`, `data/`, `\*.pt`, `\*.ts`, `\*.safetensors` stay gitignored.

4\. Do not hardcode class labels, image sizes, or normalization values. Read them from each model's `meta.json`.

5\. Stub models must be marked. Every API response that used a stub sets `is\_stub: true`. Never present stub output as a real prediction.

6\. Do not add libraries beyond Architecture.md without asking.

7\. If a requirement is ambiguous or conflicts with the docs, stop and ask. Do not guess.

8\. Do not claim a feature works until you ran it. Report what you ran and what you did not.



\## Environment

\- Windows, Python 3.11, no GPU. CPU inference only.

\- Backend in `backend/`, virtual environment in `backend/.venv`.

\- Frontend in `frontend/`, Vite + React + TypeScript + Tailwind.



\## Task end format

End every task with: files changed, commands you ran, test results, known gaps. Keep it under 15 lines.



\## Code style

\- Type hints on all backend functions. Pydantic models for all request and response bodies.

\- Small modules. No file over 300 lines.

\- Every backend feature gets at least one pytest test.

