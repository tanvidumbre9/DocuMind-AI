\# DocuMind AI: Context



\## What it is

A web app that accepts a document image (receipt, invoice, form, letter, and so on), classifies the document type, runs OCR, extracts key fields (company, date, address, total), and stores results in a history.

It is a Deep Learning course project. The app also exposes a model comparison view.



\## Scope decisions

\- Training happens outside this repo workflow, in Google Colab. The app only loads trained artifacts.

\- Database: SQLite (not PostgreSQL).

\- OCR: RapidOCR (ONNX Runtime). It uses PaddleOCR's models and runs on Windows CPU.

\- No LayoutLMv3.

\- Experiment tracking: MLflow runs exist in Colab only. The app shows metrics from `meta.json`.



\## Models the app serves

\- Image classifiers: mlp, cnn, resnet18 (TorchScript).

\- Text classifier: DistilBERT on OCR text.

\- Extractor: DistilBERT token classifier (NER) with labels COMPANY, DATE, ADDRESS, TOTAL.

RNN, LSTM, and GRU are course experiments. The app only lists their metrics. It does not run them.



\## Document classes (8)

budget, email, form, invoice, letter, memo, resume, scientific publication.

Always read the class list from `meta.json`.



\## Stub mode

Until real weights are in `models/`, the app runs with MODEL\_MODE=stub. Stub models return deterministic fake output and set `is\_stub: true`.

Real mode (MODEL\_MODE=real) loads from `models/`.



\## Quality bar

One-day MVP. A working, polished frontend. Honest separation of implemented and planned features.

