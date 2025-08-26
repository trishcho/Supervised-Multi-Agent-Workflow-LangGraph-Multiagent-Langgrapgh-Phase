# Supervised Multi-Agent Workflow (LangGraph) — **Multiagent Langgrapgh Phase**

![LangGraph](https://img.shields.io/badge/LangGraph-multiagent-blue)
![OpenAI](https://img.shields.io/badge/OpenAI-gpt--4o--mini-412991?logo=openai)
![Tavily](https://img.shields.io/badge/Tavily-Search-orange)
![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python)

A supervised **multi-agent** system built with **LangGraph** that routes user requests through specialized agents:
- **Supervisor** (routing + task typing)
- **Researcher** (web facts via **Tavily**)
- **Coder** (deterministic execution via **PythonREPLTool**)
- **Validator** (task-aware finishing: numeric vs informational)

This repository contains a **notebook-first** implementation that streams per-node updates, uses **meta tags** to avoid bad loops, and enforces safe Python execution.

---

## Author

**Trishul Chowdhury**

## ✨ Key Features

- **Supervisor-routed graph** (Enhancer → Researcher → Coder → Validator)
- **Task typing**: `numeric` vs `informational` (controls Validator criteria)
- **Researcher** uses **Tavily**; outputs concise facts + source URLs
- **Coder** generates/executes **pure Python**, returns only the final result
- **Validator** finishes when the answer meets the task type’s requirements
- **Loop guards** + **recursion limits** to prevent runaway ReAct loops
- **Streaming debug**: prints node-by-node updates (great for demos)

---

## 🧠 Architecture


User
└─► Supervisor (decides route + task_type)
├── Enhancer (optional: refine prompt)
├── Researcher (Tavily → factual summary + links)
└── Coder (PythonREPLTool → compute/print result)
↓
Validator (finish if criteria met; else back to Supervisor)


**Task types**
- `numeric` → Validator requires a **plain integer** (e.g., Fibonacci → `6765`)
- `informational` → Validator accepts a **concise factual summary** with at least one **source URL**

---

## 📦 Project Layout (suggested)


multiagent-langgraph/
├─ notebooks/
│ └─ multiagent_workflow.ipynb # end-to-end demo with streaming
├─ src/
│ └─ graph.py # optional: same logic as a script/module
└─ .env # keys for OpenAI / Tavily (not committed)


---

## ⚙️ Setup

### 1) Python & deps

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -U pip

pip install \
  langgraph langchain langchain-openai langchain-community \
  langchain-experimental python-dotenv \
  tavily-python requests



2) Environment

Create .env at repo root:

OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...


pip install -U certifi
export SSL_CERT_FILE=$(python -c "import certifi; print(certifi.where())")
export REQUESTS_CA_BUNDLE=$SSL_CERT_FILE


Run the Notebook

Open notebooks/multiagent_workflow.ipynb and run all cells.
Two quick demos are included:

Informational: “Weather in Chennai” → Supervisor → Researcher → Validator → END

Numeric: “Give me the 20th Fibonacci number” → Supervisor → Coder → Validator → END

You’ll see streaming lines like:

--- Workflow Transition: Supervisor → RESEARCHER ---
--- Workflow Transition: Coder → Validator ---
--- Validator: task_type=informational route=researcher latest_speaker=researcher ---


tests = [
  "Give me the 10th Fibonacci number",
  "Give me the 20th Fibonacci number",
  "Sum of integers from 1 to 100",
  "Weather in Chennai",
  "Who is the CEO of OpenAI?",
]
for q in tests:
    out = app.invoke({"messages":[("user", q)]}, config={"recursion_limit": 8})
    last = out["messages"][-1]
    print(f"\nQ: {q}\nA: {last.content}  (from: {getattr(last,'name','unknown')})")


Troubleshooting

401 Unauthorized (Tavily) → check TAVILY_API_KEY and that your kernel/venv actually loaded .env.

macOS SSL errors → update certs (pip install -U certifi) and set SSL_CERT_FILE/REQUESTS_CA_BUNDLE.

GraphRecursionError → increase per-call config={"recursion_limit": 8} and keep tool lists minimal per node.

Python tool safety → PythonREPLTool executes arbitrary code. We constrain prompts and block imports/IO; keep running in a trusted/local environment.