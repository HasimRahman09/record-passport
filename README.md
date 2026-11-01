# Record Passport

Our entry to the RIMPA Global Hackathon 2026, where we were finalists. The challenge was how
records and information management could help organisations deal with bias and misinformation
in AI output.

Our answer: judge an AI-assisted document by its evidence, not by whether AI wrote it. Before
the document is filed as a record, the app

1. looks up every citation (Crossref, then OpenAlex) and says whether it was found, didn't match, or doesn't exist
2. traces each sentence back to source records and marks it supported, weak, unsupported or contradicted.
   A human-written sentence gets the same test as an AI-written one
3. files a "passport" with the record: who made it, which model, who reviewed it, and which risk tier it falls in,
   chained by hash so later edits show up
4. if a model is later found faulty, or a source record is replaced, finds every record that depended on it
   and queues the risky ones for another review

There's a live version at [record-check.vercel.app](https://record-check.vercel.app). All the data in it is made up.

![A record passport](docs/demo/skin-passport.png)

| Checking a document | Dashboard | Recall |
|---|---|---|
| ![Check](docs/demo/skin-check.png) | ![Dashboard](docs/demo/skin-dashboard.png) | ![Recall](docs/demo/skin-recall.png) |

## How it's put together

```
prototype/record-check/
  recordcheck/   the engine, a plain Python package
  api/           FastAPI around the engine (runs as Vercel functions)
  app/           Streamlit console we used on stage
  tests/         284 tests, all offline
web/             Next.js front end
docs/            decision log, version log, screenshots
```

A few choices worth knowing about:

- The rules live in the engine, not the UI. The engine refuses to file a record below its tier or
  signed off by its own author, and the API recomputes verdicts rather than trusting what it's sent.
- Claude is only used to split text into claims, and there's a rule-based fallback when there's no
  key or no network. Everything else is ordinary code, so the demo and the tests run offline.
- `EDGE_CASES.md` lists every edge case we thought of, and a test fails if it gets out of step with the code.
- SQLite locally, Postgres on Neon in production, and a separate database branch for the stage demo.

The [decision log](docs/decisions.md) has the reasoning behind most of this, day by day.

## Running it

```bash
cd prototype/record-check
uv sync
uv run pytest -q
uv run uvicorn api.main:app --reload        # API on http://127.0.0.1:8000/docs
uv run streamlit run app/streamlit_app.py   # console
```

```bash
cd web && npm install && npm run dev        # front end; expects the API on :8000
```

## Team

Three of us built it over 19–21 October 2026 for RIMPA Live in Sydney. I wrote the code, and two
technology consultants led the pitch and the records research. This repo is the product code taken
out of our team workspace, with its history kept.
