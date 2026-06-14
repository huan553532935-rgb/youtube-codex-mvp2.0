# YouTube Codex MVP

This project fetches the latest YouTube comments for `@TheValley101`, classifies them with free keyword rules, writes an Excel report, and displays a local Streamlit dashboard.

## Local Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.local.example .env.local
```

Fill `.env.local`:

```bash
YOUTUBE_API_KEY=your_youtube_data_api_key
YOUTUBE_CHANNEL_HANDLE=@TheValley101
YOUTUBE_CHANNEL_ID=
MAX_COMMENTS=50
OUTPUT_EXCEL_PATH=reports/youtube_comments_latest.xlsx
```

## Run

```bash
source .venv/bin/activate
python scripts/run_daily.py
streamlit run app.py --server.address 127.0.0.1 --server.port 8502
```

## GitHub Actions

The workflow at `.github/workflows/daily-update.yml` runs daily and can also be started manually.

Before using it, add this repository secret:

```text
YOUTUBE_API_KEY
```

The workflow generates:

```text
data/youtube_comments_latest.json
reports/youtube_comments_latest.xlsx
```

and commits those generated outputs back to the repository when they change.

## Notes

- `.env.local` is intentionally ignored.
- The MVP uses free keyword rules in `scripts/classify_comments.py`.
- A future v2 can replace keyword rules with OpenAI semantic classification.
