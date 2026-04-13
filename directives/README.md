# Directives — Layer 1: What To Do

Directives are the instruction layer of the 3-layer agent architecture. They are plain Markdown files that tell the agent **what to do** — not how to code it.

---

## What Is a Directive?

A directive is a Standard Operating Procedure (SOP) written for an AI agent. It answers:

- **Goal** — what outcome are we trying to achieve?
- **Inputs** — what information does the agent need to get started?
- **Tools/Scripts** — which execution scripts in `execution/` should be used?
- **Outputs** — what does a successful result look like?
- **Edge Cases** — what should the agent do when things go wrong?

Think of it like instructions you'd write for a capable employee who's doing this task for the first time. Clear enough to act on, detailed enough to handle the hard parts.

---

## Where Directives Fit in the Architecture

```
DIRECTIVE (you are here)
    │
    │  The agent reads this
    ▼
ORCHESTRATION (the LLM)
    │
    │  The agent calls these
    ▼
EXECUTION (scripts in execution/)
```

The agent reads the directive, then decides which execution scripts to call and in what order. The directive is the contract between human intent and agent behavior.

---

## How to Write a Directive

Each directive should cover these sections:

### 1. Goal
One or two sentences describing the desired outcome.

> "Scrape the list of URLs provided and return a structured summary of each page's content."

### 2. Inputs
What the agent will receive before starting.

> - A list of URLs (plain text, one per line)
> - Optional: a target output format (JSON, CSV, Markdown)

### 3. Execution Tools
Which scripts in `execution/` handle this task. Be explicit.

> - Use `execution/scrape_single_site.py` for each URL
> - Use `execution/summarize_content.py` to process the scraped text

### 4. Outputs
What the agent should return when done.

> - A Markdown file with one section per URL
> - Each section includes: URL, page title, 3-sentence summary

### 5. Edge Cases
What the agent should do when things don't go as planned.

> - If a URL returns a 404, skip it and log the failure
> - If scraping times out after 10 seconds, retry once then skip
> - If summarization fails, return the raw scraped text instead

---

## Living Documents

Directives are not set-and-forget. They should be updated whenever the agent learns something new:

- An API has rate limits that weren't documented → update the directive
- A script fails on a specific input type → add it to edge cases
- A better approach is discovered → update the tools/scripts section

When the agent self-anneals (fixes a broken script and learns from the error), the directive gets updated so the whole system improves.

---

## Naming Convention

Use a numeric prefix to suggest execution order or grouping:

```
directives/
├── 00_workflow_example.md     ← Overview / reference example
├── 01_scrape_websites.md
├── 02_process_data.md
└── 03_generate_report.md
```

Numbers don't have to be sequential — they're just for organization.

---

## What Directives Are Not

- Not code — directives never contain scripts or implementation details
- Not prompts — they're instructions for the agent, not inputs to a model
- Not rigid specs — they're living documents that evolve with the system

---

## Example Directive Structure

```markdown
# Task Name

## Goal
What we're trying to achieve.

## Inputs
- Input 1
- Input 2

## Execution Tools
- `execution/script_name.py` — what it does
- `execution/other_script.py` — what it does

## Outputs
- Expected output format and contents

## Edge Cases
- Scenario 1 → how to handle it
- Scenario 2 → how to handle it
```

Start here, keep it simple, and add detail as you encounter real-world complexity.
