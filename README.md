# Solar Farm 

An agentic AI system that manages a solar power plant end-to-end — detecting faulty panels, forecasting energy production, monitoring weather, and recommending maintenance — built on a LangGraph state-graph workflow with real tool calling and multi-agent coordination.

## Program

Developed as part of the **"Advanced Agentic AI Systems Engineering"** program by SDAIA Academy.

Trainer: Mohammed Albeladi

## Team
- Rehaf Ismail Alfaleh
- Sarah Abdulaziz Alkhudhiri
- Raneem Abdullah Alsheddi
- Hajer Adel Almejel

*Role assignments below follow the rubric-based split the team agreed on — confirm/adjust the names against each role before submitting.*

## Overview

Solar Farm AI continuously assesses the health and output of a solar power plant through a coordinated set of specialized agents. A Planner agent decomposes an incoming task into a plan, then dedicated agents check weather conditions, analyze panel telemetry for faults, forecast expected energy output, and recommend maintenance actions where needed — all coordinated through a shared state graph with real branching and retry logic, and reviewed by a self-critique agent before the final report is produced.

## Features

- Plan-and-Execute task decomposition via a Planner agent
- Real tool-calling agents using an explicit **Thought → Action → Observation** (ReAct) loop, not hardcoded outputs
- Live weather + solar irradiance data from the Open-Meteo API
- Computed fault detection over simulated panel sensor telemetry
- Data-driven energy forecasting with a real confidence score
- Prioritized, human-readable maintenance recommendations
- Conditional/branching graph edges (fault detected → maintenance route; no fault → skip ahead)
- Conditional retry loop on low-confidence forecasts, with a capped retry count
- Reflexion-style self-critique of the final report before completion
- Short-term memory carried across each agent's reasoning steps

## Tech Stack

- Python
- Google Colab (primary development environment)
- LangGraph (StateGraph orchestration)
- Requests (Open-Meteo weather API)
- FastAPI *(Production Readiness — backend/persistence team)*
- SQLite / Redis *(Production Readiness — persistence)*
- Docker / docker-compose *(Production Readiness — deployment artifact)*
- Gradio *(dashboard/demo — Security & Documentation team)*
- LangSmith or equivalent *(observability — Security & Documentation team)*
- GitHub

## Architecture

```
Task
  ↓
Planner Agent            (decomposes task into a plan)
  ↓
Weather Agent            (real weather + irradiance via Open-Meteo)
  ↓
Panel Analysis Agent     (computed fault detection over sensor telemetry)
  ↓
 ┌─ fault detected? ──┐
 │                    │
Maintenance Agent      │
 │                    │
 └────────┬───────────┘
          ↓
Energy Prediction Agent  (forecast + confidence score)
          ↓
   confidence < threshold? ──yes──→ retry Energy Prediction Agent (capped)
          │no
          ↓
Aggregator               (compiles final report)
          ↓
Review Agent             (Reflexion — critiques report for inconsistencies)
          ↓
       Final Report
```

**Reasoning patterns implemented:**
- **Plan-and-Execute** — the Planner agent breaks the task into an ordered plan before execution begins.
- **ReAct** — each tool-calling agent (Weather, Panel Analysis, Energy Prediction, Maintenance) wraps its tool call in an explicit Thought → Action → Observation loop with short-term memory of its own steps.
- **Reflexion / self-critique** — the Review agent inspects the final report for inconsistencies (e.g. a maintenance flag paired with a low-confidence forecast) before the workflow ends.

**State**: A shared `SolarState` (TypedDict) flows through every node — each node reads relevant fields and writes its results back, rather than agents passing isolated prompts to each other.

**Coordination strategy**: Centralized/coordinator — the graph itself, entered through the Planner, sequences and routes every other agent. There is no peer-to-peer negotiation between agents.

**Branching and retry (Graph-Based Orchestration)**:
- Conditional edge after panel analysis: routes to the Maintenance agent if any panel group is flagged as faulty, otherwise proceeds directly to energy forecasting.
- Conditional edge after energy forecasting: if the forecast's confidence score falls below threshold, loops back to re-run the Energy Prediction agent (capped at 3 attempts) before continuing.


## Prerequisites & Installation

- Python 3.10+
- A Groq or OpenAI API key (for LLM-driven planning and review, once wired in)
- No API key required for weather data (Open-Meteo is free and keyless)

Install required libraries:

```bash
pip install -q langgraph langchain langchain-openai requests
```

## Configuration & Secrets

If using an LLM provider for the Planner or Review agents, store the key via Colab Secrets:

```python
from google.colab import userdata
import os
os.environ["OPENAI_API_KEY"] = userdata.get("OPENAI_API_KEY")
```

Running locally, use a `.env` file instead (never commit this file — see `.gitignore`):

```
OPENAI_API_KEY=your_api_key_here
```

```python
import os
from dotenv import load_dotenv
load_dotenv()
os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY")
```

## How to Run

1. Open the notebook in Google Colab.
2. Run the setup/pip-install cell first.
3. Run the state schema, node function, and tool function cells in order.
4. Run the graph-build cell to compile the LangGraph `StateGraph`.
5. Run the main invocation cell (`app.invoke(...)`) to execute a full pass — this produces the final report and prints each agent's Thought/Action/Observation trace.
6. Run the retry-evidence cell to confirm the confidence-based retry logic fires under a forced low-confidence scenario.
7. For a fully clean verification, use Runtime → Restart session and run all to confirm the notebook runs end-to-end with no leftover session state.

## References

- SDAIA Academy on GitHub: https://github.com/SDAIAAcademy
