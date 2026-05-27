# Trip Splitter

A Streamlit web application for managing shared expenses across group trips. Supports multiple trips, flexible participant management, category-based expense tracking, and automatic settlement optimisation.

## Features

### Trip Management
- Create and name multiple independent trips, each with their own participant list and expense history
- Per-trip participant management with add support
- Configurable expense categories per trip

### Expense Recording
- Log expenses with payer, amount, description, and category
- Exclude specific participants from individual expense splits
- Timestamp auto-recorded on submission
- Edit and delete existing expense entries

### Settlement Engine
- Computes net balance per participant across all recorded expenses
- Optimised settlement algorithm minimises the number of transactions required to clear all debts
- Displays clear pairwise settlement instructions

### Analytics & Reporting
- Per-person summary: total paid, fair share owed, and net balance
- Category-wise expense breakdown (pie chart via matplotlib)
- Day-wise expense log with per-day category visualisation
- Export raw expenses and per-person summary to CSV

## Tech Stack

| Component | Technology |
|---|---|
| App Framework | Streamlit |
| Database | MongoDB (pymongo) |
| Data Processing | pandas |
| Visualisation | matplotlib |
| Configuration | TOML (Streamlit secrets) |
| CLI Interface | Typer (optional) |

## Prerequisites

- Python 3.9+
- MongoDB instance (local or Atlas)

## Setup

```bash
pip install -r requirements.txt
```

Configure the MongoDB connection in `.streamlit/secrets.toml`:

```toml
[mongo]
uri     = "mongodb+srv://<user>:<password>@<cluster>.mongodb.net/"
db_name = "trip_splitter"
```

For local development without Streamlit Cloud, the app falls back to environment variables (see `src/trip_splitter/config.py`).

## Running the Application

```bash
streamlit run src/trip_splitter/app.py
```

Opens at `http://localhost:8501`.

## Project Structure

```
src/
└── trip_splitter/
    ├── app.py        # Main Streamlit application
    ├── cli.py        # Optional Typer CLI interface
    ├── config.py     # Configuration loading (Streamlit secrets / env)
    └── utils.py      # Balance computation and settlement optimisation
```

## Settlement Algorithm

The settlement engine uses a greedy optimisation approach:

1. Compute each participant's net balance: `total_paid − fair_share`
2. Separate participants into creditors (positive balance) and debtors (negative balance)
3. Iteratively match the largest debtor against the largest creditor, reducing both until all balances reach zero

This approach minimises the total number of required transactions compared to naive pairwise settlement.

## Database Structure

Trip configurations are stored in a shared `Trip_names` collection:

```json
{
  "trip_name": "string",
  "participants": ["string"],
  "categories":  ["string"],
  "created_at":  "ISO timestamp"
}
```

Each trip uses a dedicated MongoDB collection named after the trip. Expense documents within those collections:

```json
{
  "type":        "expense",
  "paid_by":     "string",
  "amount":      "number",
  "description": "string",
  "category":    "string",
  "included":    ["string"],
  "timestamp":   "YYYY-MM-DD"
}
```

## License

MIT
