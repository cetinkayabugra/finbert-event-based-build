# FinBERT Financial Sentiment Research Tool — Build Specification

## Role

Act as a senior software engineer building a **research-oriented financial sentiment analysis system** for an academic dissertation.

The goal is **not** to build a trading bot or investment recommendation engine. The goal is to build a **clear, reproducible, evidence-driven research pipeline** that ingests financial news, applies FinBERT-based sentiment analysis, constructs structured sentiment signals, and evaluates the relationship between sentiment and market behaviour using **event-based sampling**.

The system must be simple, academically defensible, modular, and realistic to implement.

---

## Project Objective

Build a Python-based pipeline that:

1. Collects financial news headlines/articles around selected real-world events.
2. Applies a domain-specific transformer model such as **FinBERT** for sentiment classification.
3. Converts sentiment outputs into numerical scores.
4. Aggregates those scores into daily event-window sentiment indices.
5. Collects corresponding historical stock price data.
6. Computes market returns and optional lagged returns.
7. Evaluates the relationship between sentiment and market behaviour using:
   - Pearson correlation
   - lag analysis
   - descriptive comparison
8. Produces outputs suitable for dissertation evidence:
   - CSV datasets
   - plots
   - summary tables
   - reproducible methodology

---

## Research Framing

This project is an **academic research tool**.

It must:
- support analysis of whether sentiment is associated with short-term market behaviour
- remain transparent and interpretable
- avoid overclaiming predictive power
- avoid presenting outputs as financial advice
- prioritize reproducibility over feature bloat

This is a **research pipeline**, not a productized quant platform.

---

## Core Design Principles

- Keep the implementation **simple and modular**
- Optimize for **clarity and reproducibility**
- Prefer **clean batch workflows** over real-time complexity
- Make it easy to run event studies repeatedly
- Save all intermediate data artifacts
- Use configuration files instead of hardcoding events and tickers
- Build outputs that can be directly inserted into a dissertation

---

## What We Discussed: Correct Research Strategy

Because NewsAPI historical access is limited and expensive, the system must use **event-based real sampling** rather than attempting broad random historical scraping.

### Event-Based Sampling Approach

Instead of gathering arbitrary news over long time horizons, the system must focus on a small number of **high-impact, real-world financial events**.

For each event:
- define the company/ticker
- define the event date
- define an event window such as:
  - `T-5` to `T+5`
  - or `T-3` to `T+3`
- collect relevant news published within that window
- run sentiment analysis on the collected text
- aggregate daily sentiment
- compare against stock price behaviour in the same window

This approach is stronger academically because it captures periods of heightened market relevance and gives interpretable evidence.

---

## Recommended Initial Events

Implement the system so events are configurable. Provide sample seed events such as:

1. NVIDIA earnings event
2. Tesla pricing / earnings event
3. Apple demand / supply chain event

These are examples only. The system must allow the researcher to change events easily.

---

## Common Mistakes to Avoid

The system and codebase must explicitly avoid these mistakes.

### Research Mistakes
- Do not collect random unrelated news with no event rationale.
- Do not mix multiple unrelated companies in one event study.
- Do not omit the exact event date.
- Do not skip defining the event window.
- Do not claim correlation proves causation.
- Do not describe outputs without quantifying them.
- Do not build conclusions that are stronger than the evidence.
- Do not rely on one article or one day of data.
- Do not use sentiment alone without linking it to market data.

### Engineering Mistakes
- Do not hardcode data paths everywhere.
- Do not tightly couple ingestion, modeling, analysis, and plotting.
- Do not hide intermediate transformations.
- Do not discard raw data after cleaning.
- Do not build a fragile scraper-first design if APIs/datasets are available.
- Do not create unnecessary front-end complexity.
- Do not overengineer authentication, dashboards, or cloud deployment.

### Dissertation Mistakes
- Do not present this as an investment system.
- Do not ignore limitations such as API restrictions and sample size.
- Do not omit methodological justification.
- Do not omit reproducibility considerations.
- Do not fail to save evidence artifacts such as charts and tables.

---

## Minimum Viable System Scope

Build a research tool with these capabilities:

### 1. Event Configuration
A structured way to define:
- event name
- company name
- ticker
- event date
- event window start offset
- event window end offset
- keywords

Example shape:

```json
{
  "event_name": "NVIDIA Qx Earnings",
  "company": "NVIDIA",
  "ticker": "NVDA",
  "event_date": "2025-02-20",
  "window_before_days": 5,
  "window_after_days": 5,
  "keywords": ["NVIDIA", "earnings", "AI chips"]
}
```

### 2. News Ingestion
Support one or more of the following:
- GNews API
- Alpha Vantage news endpoints
- manually curated CSV input
- optional scraping adapters for approved public sources

The ingestion layer must normalize records into a unified schema.

### 3. Text Preparation
For each article/headline:
- normalize text
- handle missing values
- optionally combine title + description/content
- remove duplicates

### 4. Sentiment Inference
Use FinBERT to classify text into:
- positive
- neutral
- negative

Also output model confidence scores if available.

### 5. Numerical Sentiment Conversion
Map classes into numerical values, for example:
- positive = `+1`
- neutral = `0`
- negative = `-1`

Also support probability-weighted sentiment if feasible.

### 6. Daily Aggregation
Aggregate all articles per day into:
- article count
- mean sentiment score
- weighted sentiment score
- positive/neutral/negative proportions

### 7. Market Data Retrieval
Use Yahoo Finance or equivalent source to get:
- daily open
- daily close
- adjusted close if available
- daily returns

### 8. Event Study Dataset
Create a merged per-day dataset with:
- date
- event relative day (`-5`, `-4`, ..., `0`, ..., `+5`)
- ticker
- close price
- return
- lagged return(s)
- mean sentiment
- article volume

### 9. Analysis
Implement:
- Pearson correlation between sentiment and same-day return
- Pearson correlation between sentiment and next-day return
- optional lag correlations for 1–3 days
- descriptive summaries

### 10. Visualization
Generate:
- line chart of daily sentiment over event window
- line chart of stock price over event window
- optional scatter plot of sentiment vs return
- saved PNG outputs for dissertation inclusion

### 11. Evidence Outputs
Save:
- raw news CSV
- cleaned news CSV
- daily aggregated sentiment CSV
- merged event study CSV
- statistical summary TXT/MD
- charts as PNG

---

## Functional Requirements

### Event Management
- load events from `config/events.yaml` or `config/events.json`
- validate date formats and required fields
- support running one or many events in a batch

### News Data Collection
- query by company name and keywords
- restrict results to the event window
- save raw responses for traceability
- include publication timestamps and source metadata

### Deduplication
- remove duplicate articles/headlines by URL and normalized text
- log number of removed duplicates

### Sentiment Scoring
- produce sentiment label
- produce numerical sentiment score
- optionally store raw model probabilities

### Market Data
- align trading dates with publication dates
- handle non-trading days gracefully
- document how weekend/holiday news is aligned

### Analysis
For each event, generate:
- event summary
- article counts
- average sentiment
- price movement description
- Pearson correlation values
- lagged correlation values

### Reporting
Generate a concise markdown report per event summarizing:
- event metadata
- number of articles
- sentiment trends
- stock movement
- correlations
- key interpretation
- limitations

---

## Non-Functional Requirements

- Python 3.11+
- reproducible environment via `requirements.txt` or `pyproject.toml`
- modular package structure
- clear logging
- robust error handling
- deterministic saved outputs where possible
- easy to run locally

---

## Proposed Project Structure

```text
finbert-research-tool/
├── README.md
├── pyproject.toml
├── requirements.txt
├── config/
│   ├── events.yaml
│   └── settings.yaml
├── data/
│   ├── raw/
│   ├── processed/
│   └── market/
├── outputs/
│   ├── charts/
│   ├── reports/
│   └── tables/
├── src/
│   ├── main.py
│   ├── pipeline.py
│   ├── config_loader.py
│   ├── models/
│   │   └── schemas.py
│   ├── ingestion/
│   │   ├── base.py
│   │   ├── gnews_client.py
│   │   ├── alpha_vantage_client.py
│   │   └── csv_loader.py
│   ├── preprocessing/
│   │   └── clean_news.py
│   ├── sentiment/
│   │   └── finbert_analyzer.py
│   ├── market/
│   │   └── price_fetcher.py
│   ├── analysis/
│   │   ├── aggregator.py
│   │   ├── correlation.py
│   │   └── event_study.py
│   ├── reporting/
│   │   ├── charts.py
│   │   └── report_writer.py
│   └── utils/
│       ├── dates.py
│       ├── io.py
│       └── logging.py
└── tests/
```

---

## Data Schema Expectations

### Raw News Record
Each raw news item should capture at minimum:
- `source`
- `title`
- `description`
- `content` if available
- `url`
- `published_at`
- `company`
- `ticker`
- `event_name`

### Cleaned News Record
Add:
- `normalized_text`
- `sentiment_label`
- `sentiment_score`
- `positive_prob`
- `neutral_prob`
- `negative_prob`

### Daily Aggregate Record
Add:
- `date`
- `article_count`
- `mean_sentiment`
- `weighted_sentiment`
- `positive_ratio`
- `neutral_ratio`
- `negative_ratio`

### Event Study Record
Add:
- `date`
- `relative_day`
- `ticker`
- `close`
- `return`
- `return_lag_1`
- `return_lag_2`
- `mean_sentiment`
- `article_count`

---

## Analytical Logic

### Pearson Correlation
The system must compute Pearson correlation to quantify the linear relationship between:
- daily sentiment and same-day returns
- daily sentiment and next-day returns
- optionally sentiment and lagged returns over 1–3 days

This provides **quantitative evidence of association**, not proof of causation.

### Lag Analysis
Lag analysis is required because market reaction may not occur on the same day as publication.

At minimum:
- `sentiment_t` vs `return_t`
- `sentiment_t` vs `return_t+1`

Optionally:
- `sentiment_t` vs `return_t+2`
- `sentiment_t` vs `return_t+3`

### Interpretation Rules
The reporting layer must avoid overclaiming:
- say “association” or “relationship”
- do not say “proves sentiment causes price movement”
- include note when sample sizes are small

---

## Implementation Guidance

### Keep It Simple
Do not build:
- authentication
- user accounts
- real-time streaming
- broker integrations
- recommendation features
- cloud microservices unless explicitly needed later

### Build for Research Reproducibility
Every pipeline run should:
- read config
- fetch/store data
- score/store sentiment
- aggregate/store daily outputs
- merge/store market data
- analyze/store results
- generate report/store plots

### Logging
Log:
- event being processed
- date window
- number of retrieved articles
- duplicates removed
- sentiment scoring completion
- market rows fetched
- correlations computed
- output locations

---

## Suggested Tech Stack

- **Python**
- **pandas**
- **numpy**
- **transformers**
- **torch**
- **yfinance**
- **matplotlib**
- **scipy**
- **pyyaml**
- **requests**

Optional:
- **beautifulsoup4** for scraping adapters
- **pydantic** for schema validation

---

## Processing Flow

```text
Load Events
   ↓
Resolve Event Windows
   ↓
Collect News Data
   ↓
Normalize and Clean Text
   ↓
Run FinBERT Sentiment Inference
   ↓
Convert Labels to Numerical Scores
   ↓
Aggregate Daily Sentiment
   ↓
Fetch Historical Market Data
   ↓
Merge Sentiment + Price Data
   ↓
Compute Correlation and Lag Analysis
   ↓
Generate Charts and Markdown Reports
```

---

## Output Requirements for Dissertation Use

For each event, produce:

1. A clean table showing:
   - date
   - relative day
   - article count
   - mean sentiment
   - close price
   - return

2. A markdown summary containing:
   - event description
   - data collection window
   - article count
   - average sentiment trend
   - same-day correlation
   - lagged correlation
   - interpretation
   - limitations

3. Charts:
   - sentiment over time
   - price over time
   - scatter plot for sentiment vs return

4. A final combined summary across events.

---

## Error Handling Requirements

The implementation must handle:
- empty news responses
- malformed timestamps
- duplicate articles
- missing article text
- non-trading days
- unavailable price data
- API failures and rate limits

If an event lacks sufficient data:
- skip gracefully
- write a warning
- still generate a report explaining insufficiency

---

## Testing Expectations

Add lightweight tests for:
- event window generation
- sentiment score mapping
- daily aggregation logic
- merge logic between sentiment and market data
- lag calculation correctness

---

## README Expectations

The generated repository README should include:
- project purpose
- academic framing
- architecture summary
- setup instructions
- how to define events
- how to run the pipeline
- output descriptions
- limitations
- disclaimer that it is not financial advice

---

## Example Run Commands

```bash
python -m src.main --event "NVIDIA Qx Earnings"
python -m src.main --all-events
```

Optional:
```bash
python -m src.main --config config/events.yaml
```

---

## Deliverable Standard

The final implementation should feel:
- academically credible
- senior-engineer structured
- easy for a dissertation examiner to understand
- easy for a student to run and explain

It should produce evidence such as:
- “During the selected event window, aggregated sentiment showed a moderate positive association with next-day return.”
- backed by saved datasets, charts, and reproducible code

---

## Final Build Instruction

Build this as a clean, maintainable, modular Python research repository that implements **event-based real-world sampling** for financial sentiment analysis using FinBERT.

Do not overcomplicate the system.
Do not introduce unnecessary product features.
Do not ignore data limitations.
Do not make causal claims from correlation.

Prioritize:
- clarity
- methodological defensibility
- evidence generation
- dissertation readiness
- simplicity with rigor
