\# Architecture



\## Stack

Backend: FastAPI, Uvicorn, SQLAlchemy 2, SQLite, Pydantic v2, python-multipart, Pillow, numpy, rapidocr-onnxruntime, torch + torchvision (CPU), transformers, huggingface\_hub, pytest, httpx.

Frontend: React 18, TypeScript, Vite, Tailwind CSS, React Router, no state library.



\## Folders

backend/app/{main.py, config.py, db.py, models\_db.py, schemas.py, routers/, services/}

backend/tests/

frontend/src/{api/, pages/, components/}

models/            (gitignored, downloaded artifacts)

uploads/           (gitignored, stored document images)

scripts/download\_models.py

ml/                (off-limits)



\## Config (env vars, with defaults)

MODEL\_MODE=stub | real (default stub)

MODELS\_DIR=models

UPLOADS\_DIR=uploads

DB\_URL=sqlite:///./documind.db

MAX\_UPLOAD\_MB=10

OCR\_MAX\_SIDE=1600



\## Model artifact contract

models/<name>/ where name is one of: img\_mlp, img\_cnn, img\_resnet18, text\_distilbert, ner\_distilbert, seq\_rnn, seq\_lstm, seq\_gru.

Every folder has meta.json.



\### Image models (img\_\*)

Files: model.ts (TorchScript), model\_state.pt, meta.json.

meta.json keys: task, model, classes (list), input {size, channels, mean, std, resize, color}, params\_total, train\_seconds, metrics {test\_accuracy, test\_macro\_f1, best\_val\_accuracy}.

Preprocessing, exactly: open image, convert to grayscale; if channels is 3, replicate to 3 channels; resize to (size, size) bilinear, no aspect preservation; to tensor in \[0,1]; normalize with mean and std; add batch dim.

Output: logits. Apply softmax. Class order = meta.classes.



\### Text classifier (text\_distilbert)

Files: hf/ (model and tokenizer), meta.json with classes, max\_len.

Input: full OCR text string. Tokenize with truncation, max\_length=meta.max\_len. Softmax over logits. Class order = meta.classes.



\### NER (ner\_distilbert)

Files: hf/, meta.json with labels (BIO list such as O, B-COMPANY, I-COMPANY, B-DATE, I-DATE, B-ADDRESS, I-ADDRESS, B-TOTAL, I-TOTAL), chunk\_words, max\_len.

Input: list of words from OCR. Split into chunks of chunk\_words words. Tokenize each chunk with is\_split\_into\_words=True, truncation, max\_length=max\_len. Use the label of each word's first sub-token. Merge consecutive B/I tags of the same type into entities. For each field, return the first entity found; for TOTAL, return the last entity found.



\### Experiment models (seq\_\*)

Only meta.json is read, for the comparison page.



\## OCR service

Interface: ocr(image) -> {text, words: \[{text, box: \[x1,y1,x2,y2], conf}], seconds}.

RapidOCR returns line-level boxes. Split each line into words on whitespace. Each word keeps its line box.

Resize so the longest side is at most OCR\_MAX\_SIDE before OCR, and scale boxes back to original coordinates.

Words are in reading order (top to bottom, left to right), as returned by RapidOCR.



\## Database (SQLite)

Table documents: id (int pk), filename, stored\_path, created\_at, classifier\_used, label, confidence, ocr\_text, result\_json (full response JSON), is\_stub (bool).



\## API

GET  /api/health -> {status, mode}

GET  /api/models -> list of {name, task, available, is\_stub, meta\_summary, metrics}

POST /api/analyze  (multipart: file; optional form field classifier, default img\_resnet18) -> AnalysisResult

POST /api/compare  (multipart: file) -> {results: \[{model, label, confidence, probs, seconds, is\_stub}]}

GET  /api/documents -> list (id, filename, label, confidence, created\_at)

GET  /api/documents/{id} -> AnalysisResult

GET  /api/documents/{id}/image -> image bytes

DELETE /api/documents/{id}



AnalysisResult:

{ id, filename, created\_at, is\_stub,

&#x20; classification: {model, label, confidence, probs: {label: float}},

&#x20; ocr: {text, words: \[{text, box, conf}], seconds, image\_width, image\_height},

&#x20; extraction: {company, date, address, total: each {value: string} or null},

&#x20; timings: {ocr, classify, extract, total} }



Errors: JSON {detail}. 415 for non-image upload, 413 for too large, 422 for bad classifier name.



\## Frontend pages

/          Analyze: drag-and-drop upload, classifier dropdown, results.

/history   Table of past documents, click to open, delete.

/document/:id  Result view: image with word boxes overlay toggle, classification bars, extracted fields, raw OCR text.

/models    Model cards with metrics from /api/models, and a table comparing all models.

Show a visible "Stub mode" banner whenever any response has is\_stub true.

