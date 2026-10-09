# Draftwise — ML writing companion

A responsive predictive-writing app for machine-learning documentation. It provides inline suggestions, domain-aware methodology checks, local draft persistence, and Markdown export.

## Requirements

- Node.js 18 or newer
- npm

## Run locally

```bash
npm install
npm run dev
```

Open the Vite URL printed by the client process (usually `http://localhost:5173`). The Express API runs on port 3001, and Vite proxies `/api` requests to it.

To build and run the production version:

```bash
npm run build
npm start
```

The production server serves the built app and API on port 3001. Optional server settings are documented in `.env.example`; copy it to `.env` to configure `PORT`. The bundled generator is local and does not need a model API key. No draft text is sent to a third-party service.

## Features

- Debounced, inline ghost-text suggestions; press **Tab** to accept, **Esc** to dismiss, and **Alt+]** to cycle alternatives.
- Data, Models, Evaluation, and Auto-detect domains, with phrase, sentence, or paragraph suggestions.
- Technical report, README, research paper, and notebook-comment styles.
- Adjustable alternative count and creativity.
- Methodology checks for test-set tuning, accuracy on imbalanced data, and preprocessing before the split.
- Quick-start templates, copy, Markdown export, clear, light/dark mode, and automatic local draft saving.
- Suggestions use placeholders for unknown results and do not generate dataset names or citations.
- API request validation, a 4,000-character cap, rate limiting, and clear client errors.

The local bundled generator trains a small domain-specific word n-gram model from the included ML-writing corpus and uses it to rank curated, safety-conscious continuations. This keeps suggestions grounded and avoids made-up results while running without network credentials.

## Five test inputs

Use each as the editor text, then choose the matching domain if you want to bypass auto-detection.

1. **Data:** `The dataset contains 18,000 rows. We handle missing values by`  
   **Good output:** a continuation describing an imputation approach learned from training data; it should not invent a missingness rate or feature name.
2. **Leakage check:** `We impute missing values before the train/test split.`  
   **Good output:** a warning to split first and fit preprocessing on training data only.
3. **Models:** `We trained a transformer classifier with`  
   **Good output:** a continuation about validation-based selection, early stopping, or a documented training configuration; it should not invent a score.
4. **Imbalanced evaluation:** `The classes are imbalanced, so accuracy is our primary metric.`  
   **Good output:** a warning that accuracy may mislead and a suggestion to report measures such as precision, recall, or F1.
5. **Test leakage:** `We tuned the threshold using the test set.`  
   **Good output:** a warning to choose the threshold on validation data and reserve the test set for final evaluation.

## API

`POST /api/suggest`

```json
{
  "text": "We trained a classifier",
  "mode": "models",
  "length": "sentence",
  "n": 2,
  "temperature": 0.4,
  "style": "report"
}
```

Returns `{ "suggestions": ["..."], "warnings": [] }`. Supported modes: `data`, `models`, `evaluation`, `auto`; lengths: `phrase`, `sentence`, `paragraph`; styles: `report`, `readme`, `paper`, `notebook`.
