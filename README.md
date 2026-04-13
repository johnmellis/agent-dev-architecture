# Agent Development Architecture

A practical framework for building reliable, maintainable AI agent systems. The core idea: **LLMs are great at decision-making, terrible at consistency**. This architecture separates the two so each layer does what it's good at.

---

## The Problem This Solves

When an AI agent does everything itself — reading context, making decisions, calling APIs, processing data — errors compound. At 90% accuracy per step, a 5-step process succeeds only 59% of the time. Push complexity into deterministic code and let the LLM focus purely on decision-making.

---

## The 3-Layer Architecture

```
┌─────────────────────────────────────────────┐
│  Layer 1: DIRECTIVE                          │
│  directives/                                 │
│  What to do — goals, inputs, outputs         │
│  Written in plain Markdown                   │
└──────────────────┬──────────────────────────┘
                   │ reads
┌──────────────────▼──────────────────────────┐
│  Layer 2: ORCHESTRATION                      │
│  The LLM / Agent                            │
│  Decision-making — routing, error handling   │
│  Calls execution tools in the right order    │
└──────────────────┬──────────────────────────┘
                   │ calls
┌──────────────────▼──────────────────────────┐
│  Layer 3: EXECUTION                          │
│  execution/                                  │
│  How to do it — deterministic scripts        │
│  Python, APIs, file ops, data processing     │
└─────────────────────────────────────────────┘
```

### Layer 1 — Directive (`directives/`)

SOPs written in Markdown. Each directive tells the agent:
- **What** the goal is
- **What** the inputs are
- **Which** execution tools/scripts to use
- **What** the expected outputs look like
- **How** to handle edge cases

Think of directives as instructions you'd give a capable employee — clear enough to act on, without specifying every line of code.

→ See [`directives/README.md`](directives/README.md)

---

### Layer 2 — Orchestration (The Agent)

This is the LLM. Its job is **intelligent routing**, not execution:

- Read the relevant directive
- Decide which execution scripts to call and in what order
- Pass the right inputs to each script
- Handle errors and retry when things break
- Update directives when it learns something new

The agent does **not** scrape websites, call APIs, or process data directly — it delegates that to Layer 3.

---

### Layer 3 — Execution (`execution/`)

Deterministic Python scripts that do the actual work:

- API calls
- Data processing and transformation
- File operations
- Database interactions

Scripts are reliable, testable, and fast. They take explicit inputs and return predictable outputs. The agent never needs to guess what happened.

→ See [`execution/README.md`](execution/README.md)

---

## How It All Works Together

1. A user gives the agent a task (e.g., "scrape these 10 URLs and summarize them")
2. The agent reads the relevant directive in `directives/`
3. The directive specifies which script in `execution/` handles this
4. The agent calls the script with the right inputs
5. The script returns a predictable result
6. If something breaks, the agent reads the error, fixes the script, and updates the directive with what it learned

This **self-annealing loop** means the system gets stronger every time something goes wrong.

---

## Directory Structure

```
agent-dev-architecture/
├── README.md               ← You are here
├── directives/             ← Layer 1: What to do (Markdown SOPs)
│   ├── README.md
│   └── 00_workflow_example.md
└── execution/              ← Layer 3: How to do it (Python scripts)
    └── README.md
```

---

## Adapting This to Your Needs

1. **Define your tasks** — write a directive in `directives/` for each repeatable task your agent needs to perform
2. **Build your scripts** — write a script in `execution/` for each operation that needs to be reliable and deterministic
3. **Point the directive at the script** — in the directive, specify which script handles which step
4. **Let the agent orchestrate** — the LLM reads directives and calls scripts; it never hard-codes logic itself

The more you push into deterministic scripts, the more reliable your agent becomes.
