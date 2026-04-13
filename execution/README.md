# Execution — Layer 3: How To Do It

The `execution/` directory contains deterministic scripts that do the actual work. This is the engine room of the 3-layer agent architecture.

---

## What Is an Execution Script?

An execution script is a Python script that performs **one specific operation** reliably:

- Calling an external API
- Scraping a web page
- Processing or transforming data
- Reading from or writing to a database
- File operations (read, write, convert)

Scripts take explicit inputs, do exactly what they say, and return predictable outputs. They do not make decisions — that's the agent's job.

---

## Where Execution Scripts Fit in the Architecture

```
DIRECTIVE (directives/)
    │
    │  The agent reads this
    ▼
ORCHESTRATION (the LLM)
    │
    │  The agent calls these
    ▼
EXECUTION (you are here)
```

The agent calls these scripts in the order specified by the directive. The script runs, returns a result, and the agent decides what to do next.

---

## Why This Layer Exists

LLMs are probabilistic. They're good at reasoning, routing, and handling ambiguity. They are not good at consistently executing the same operation the same way every time.

Execution scripts fix that. By pushing complexity into deterministic code:

- **Reliability improves** — the same input always produces the same output
- **Errors are easier to debug** — stack traces are specific and actionable
- **The agent stays focused** — it routes and decides, it doesn't execute
- **Scripts can be tested independently** — no agent needed

---

## What Makes a Good Execution Script

### Single responsibility
Each script does one thing. If you find yourself writing a script that scrapes, summarizes, *and* saves to a database, split it into three scripts.

### Explicit inputs and outputs
Scripts should accept arguments (via command-line args, environment variables, or stdin) and return clear outputs. Avoid relying on hidden state.

### Predictable failure
When something goes wrong, scripts should fail loudly with a clear error message. The agent reads error output and uses it to self-anneal.

### No decision-making
Scripts don't decide *what* to do — they just do it. If a script contains conditional logic about which task to perform, that logic probably belongs in the directive or the agent.

---

## Self-Annealing Loop

When a script breaks, the agent:

1. Reads the error message and stack trace
2. Fixes the script
3. Tests it again
4. Updates the directive in `directives/` with what it learned

This means the system improves every time something fails. Over time, scripts become more robust and directives become more accurate.

Example: a script hits an API rate limit → the agent investigates the API docs → discovers a batch endpoint → rewrites the script to use it → tests it → updates the directive to note the rate limit behavior.

---

## Environment Variables and Secrets

Scripts should read credentials and configuration from environment variables or a `.env` file. Never hardcode secrets into scripts.

```
.env
├── API_KEY=...
├── DATABASE_URL=...
└── OUTPUT_DIR=...
```

Use a library like `python-dotenv` to load these in scripts.

---

## Naming Convention

Name scripts after what they do, not what they're part of:

```
execution/
├── scrape_single_site.py       ← fetches and parses one URL
├── summarize_content.py        ← takes text, returns a summary
├── save_to_database.py         ← writes a record to the DB
└── export_to_csv.py            ← converts data to CSV format
```

Each script name should be a verb phrase that describes its action.

---

## What Execution Scripts Are Not

- Not orchestration logic — scripts don't decide which other scripts to call
- Not directives — scripts don't define goals or describe edge cases
- Not catch-all utilities — each script should have a focused, single purpose

---

## Example Script Structure

```python
"""
scrape_single_site.py

Input:  A single URL (passed as a command-line argument)
Output: The page title and body text, printed as JSON to stdout

Usage:
    python scrape_single_site.py https://example.com
"""

import sys
import json

def scrape(url: str) -> dict:
    # ... implementation ...
    return {"url": url, "title": "...", "body": "..."}

if __name__ == "__main__":
    url = sys.argv[1]
    result = scrape(url)
    print(json.dumps(result))
```

Clear docstring, explicit input, predictable output. The agent knows exactly what to expect.
