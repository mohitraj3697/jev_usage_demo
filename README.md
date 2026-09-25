# JEV Usage Demo

A beginner-friendly example showing how to use **JEV** to run topic-level sentiment analysis on customer reviews.

---

## What is JEV?

JEV is a language model designed for structured evaluation tasks. Instead of generating free-form text, you give JEV a piece of text (the *state*) and a set of *questions* — each returning a calibrated score or probability — and it answers all of them in a single call. This makes it well-suited for sentiment analysis, content moderation, review scoring, and similar structured-output workflows.

> _Feel free to expand this section with links to the official JEV docs or paper._

---

## Prerequisites

- **Python 3.11+**
- A **TypeSafe API key** (set as `TYPESAFE_API_KEY` in a `.env` file)

### Installation

```bash
# Clone the repo
git clone https://github.com/<your-username>/jev_usage_demo.git
cd jev_usage_demo

# Install dependencies (using uv — recommended)
uv sync

# Or install with pip
pip install typesafe-sdk python-dotenv
```

Create a `.env` file in the project root:

```
TYPESAFE_API_KEY=your_api_key_here
```

---

## Project Structure

```
jev_usage_demo/
├── main.py                          # Entry point — reads CSV, runs analysis, prints summary
├── analyzer.py                      # Sends a review to JEV and extracts per-topic star ratings
├── questions.py                     # Defines the 14 JEV questions (7 topics × 2 questions each)
├── aggregation.py                   # Averages per-review ratings into an overall summary
├── synthetic_iphone16_reviews.csv   # 50 synthetic iPhone 16 reviews (demo dataset)
├── pyproject.toml                   # Project metadata and dependencies
└── .env                             # Your TYPESAFE_API_KEY (not committed)
```

---

## Usage

### 1. Run the full pipeline

```bash
python main.py
```

This will:

1. Load all 50 reviews from `synthetic_iphone16_reviews.csv`.
2. Send each review's `review_text` to JEV with 14 questions (7 topics × 2 questions).
3. Print per-review topic ratings, then an aggregated summary.

### 2. Step-by-step walkthrough

Below is a simplified version of what `main.py` does, so you can follow along:

```python
import csv
from questions import TOPICS, QUESTIONS
from analyzer import analyze_review

# Load the dataset
with open("synthetic_iphone16_reviews.csv", encoding="utf-8") as f:
    reviews = list(csv.DictReader(f))

# Analyze each review
for review in reviews:
    ratings = analyze_review(review["review_text"])
    print(f'{review["review_id"]}: {ratings}')
```

`analyze_review()` calls JEV under the hood:

```python
from typesafe_sdk import TypeSafeClient

client = TypeSafeClient()  # reads TYPESAFE_API_KEY from .env

response = client.system_one(
    model="jev-latest",
    state=review_text,       # the review to evaluate
    questions=QUESTIONS,     # 14 Noul/Score questions defined in questions.py
)
```

For each of the 7 topics (Camera, Battery, Display, Design, Performance, Build Quality, Value for Money), JEV answers two questions:

| Question type | Returns | Purpose |
|---|---|---|
| `Noul` (e.g. `camera_mentioned`) | Probability 0–1 | Is this topic discussed in the review? |
| `Score` (e.g. `camera_rating`) | Level 0–4 | How satisfied is the reviewer? (mapped to 1–5 stars) |

A topic is included in the output only if JEV's mention probability ≥ 0.5.

---

## Example Output

Per-review ratings:

```
R001 {'Camera': 4.2, 'Battery': 4.5, 'Value for Money': 4.8}
R002 {'Battery': 1.8}
R003 {'Camera': 3.4}
```

Aggregated summary:

```
3.2 / 5  based on 50 ratings
----------------------------------------
Camera           3.8 / 5  (32 reviews)
Battery          3.5 / 5  (28 reviews)
Display          4.1 / 5  (22 reviews)
Design           3.6 / 5  (18 reviews)
Performance      3.3 / 5  (14 reviews)
Build Quality    3.9 / 5  (12 reviews)
Value for Money  3.4 / 5  (16 reviews)
```

> **Note:** The numbers above are illustrative. Your actual scores will depend on the JEV model version.

---

## Synthetic Dataset

The file `synthetic_iphone16_reviews.csv` contains **50 entirely synthetic reviews** generated for demonstration purposes. They do not represent real customer opinions. The `is_synthetic` column is set to `true` for every row. **Do not use this data for real product insights.**

---

## License

This project is licensed under the [MIT License](LICENSE).
