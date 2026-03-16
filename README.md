# 🌍 War Sentiment Analyzer — YouTube Comments

A data pipeline that scrapes YouTube comments from CNN and Fox News, classifies each comment's political stance using zero-shot classification, and visualizes the results through an interactive Streamlit dashboard — covering both the Russia-Ukraine war and the Israel-Gaza conflict.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Setup & Installation](#setup--installation)
- [How to Run](#how-to-run)
- [Dashboard Pages](#dashboard-pages)
- [Notes & Limitations](#notes--limitations)
- [Ethical Disclaimer](#ethical-disclaimer)

---

## Overview

This project scrapes YouTube comments from war-related videos posted by CNN and Fox News. Each comment is classified into a political stance (e.g. Pro-Ukraine, Pro-Russia, Pro-Gaza, Pro-Israel, or Neutral) using Facebook's `bart-large-mnli` zero-shot classification model. The classified data is then visualized through a multi-page Streamlit dashboard that computes bias scores and breaks down sentiment per channel, topic, and individual video.

---

## Features

- 🕷️ Scrapes comment text, likes, and reply counts from YouTube via Selenium + BeautifulSoup
- 🧠 Zero-shot classification using `facebook/bart-large-mnli` — no training data required
- 🎯 Context-aware classification — video title is prepended to each comment before classification
- 📊 Multi-page Streamlit dashboard with Plotly charts
- ⚖️ Bias score computation per channel (Ukraine Support % minus Gaza Support %)
- 💾 All scraped and classified data saved as CSV

---

## Tech Stack

| Layer | Tools |
|---|---|
| Scraping | Python, Selenium, BeautifulSoup4 |
| Data Handling | Pandas |
| Classification | HuggingFace Transformers (`facebook/bart-large-mnli`), PyTorch |
| Dashboard | Streamlit, Plotly Express |
| Storage | CSV |

---

## Project Structure

```
PAI_Project/
│
├── finalscraper.ipynb          # Scraping pipeline (Selenium + BeautifulSoup)
├── finalanalysis.ipynb         # Zero-shot classification pipeline
├── app.py                      # Streamlit dashboard
│
├── scrapedcnn/                 # Raw scraped CSVs (one per video)
│
├── classifiedcnn/
│   ├── ru/                     # Classified Ukraine-Russia CSVs (CNN)
│   └── ip/                     # Classified Israel-Palestine CSVs (CNN)
│
├── classifiedfoxnews/
│   ├── ru/                     # Classified Ukraine-Russia CSVs (Fox News)
│   └── ip/                     # Classified Israel-Palestine CSVs (Fox News)
│
└── requirements.txt
```

---

## Setup & Installation

**1. Clone the repository**
```bash
git clone https://github.com/your-username/war-sentiment-analyzer.git
cd war-sentiment-analyzer
```

**2. Install dependencies**
```bash
pip install -r requirements.txt
```

If you don't have a `requirements.txt`, install manually:
```bash
pip install selenium beautifulsoup4 pandas transformers torch streamlit plotly
```

**3. Set up ChromeDriver**

The scraper uses a custom Chrome binary and ChromeDriver. Update these two lines in `finalscraper.ipynb` to match your local paths:
```python
options.binary_location = "D:/Program Files/chrome-win64/chrome.exe"
service = Service(r"D:\Program Files\chromedriver-win64\chromedriver.exe")
```

Download ChromeDriver from [chromedriver.chromium.org](https://chromedriver.chromium.org) — make sure the version matches your installed Chrome.

---

## How to Run

### Step 1 — Scrape Comments

Open `finalscraper.ipynb` and run all cells. When prompted, paste a YouTube video URL:

```
Enter the url: https://www.youtube.com/watch?v=...
```

The script will open Chrome, scroll through the entire comment section, and save a CSV to `scrapedcnn/` named after the video title. Repeat for each video you want to scrape.

Each CSV contains:
| Column | Description |
|---|---|
| `text` | Comment text |
| `likes` | Number of likes on the comment |
| `replies` | Number of replies |

---

### Step 2 — Classify Comments

Open `finalanalysis.ipynb` and run all cells.

- Use `classify(title)` for **Russia-Ukraine** videos → saves to `classifiedcnn/ru/`
- Use `classify2(title)` for **Israel-Gaza** videos → saves to `classifiedcnn/ip/`

When prompted:
```
Enter file name: Ukrainian soldiers have three seconds to shoot down a Russian attack drone
```

Enter the filename **without** the `.csv` extension. The model will run zero-shot classification on each comment using the video title as context and append a `predicted_stance` column.

> ⚠️ This step runs on CPU by default and may be slow for large comment sets. A GPU will significantly speed things up.

---

### Step 3 — Launch the Dashboard

```bash
streamlit run app.py
```

Make sure all classified CSVs are in the correct folders as referenced in `app.py` before launching.

---

## Dashboard Pages

| Page | Description |
|---|---|
| **Overall Stance Distribution** | 100% stacked bar chart of all stances per channel |
| **Bias Score per Channel** | Horizontal bar chart — Ukraine Support % minus Gaza Support % |
| **Likes-weighted Chart** | Total likes grouped by stance and channel |
| **Replies-weighted Chart** | Total replies grouped by stance and channel |
| **Pro-victim vs Other** | Stacked chart comparing pro-victim sentiment per channel and topic |
| **Bias Shift Line** | Line chart showing how support shifts between Ukraine and Gaza per channel |
| **Bias Score Table** | Raw bias scores with interpretation guide |
| **Individual Video Charts** | Per-video stance breakdown for every scraped video |

---

## Notes & Limitations

- YouTube loads comments dynamically via JavaScript — the scraper scrolls to the bottom of the page to load all comments, which can take several minutes per video
- `facebook/bart-large-mnli` is a general-purpose NLI model, not fine-tuned on political text — sarcasm, irony, and ambiguous comments may be misclassified
- Classification runs on CPU by default; for large datasets consider running on a GPU
- File paths in `app.py` are hardcoded — if you rename or move files, update the `fileMap` dictionary accordingly
- Only English comments are reliably classified

---

## Ethical Disclaimer

This project is purely academic. The goal is to measure observable patterns in public comment sentiment — not to advocate for any side in either conflict. Labels like "Pro-Russia" or "Pro-Israel" reflect the model's interpretation of comment text only and do not represent the views of the developers. No personal user data is stored or published.

---

## License

MIT License — free to use and adapt for educational and research purposes.
