## Setup Instructions
1. Clone the repository
2. Create a virtual environment: `python -m venv venv`
3. Activate it: `source venv/bin/activate` (or `venv\Scripts\activate` on Windows)
4. Install dependencies: `pip install -r requirements.txt`
5. Copy `.env.example` to `.env` and add your Gemini API key
6. Launch Jupyter: `jupyter notebook detective_simulator.ipynb`
7. Run all cells from top to bottom, then interact with the detective in the last cell

# 🕵️ Multi-Agent AI Detective Simulator

## The Midnight Murder at Lord Ashworth's Mansion

A LangGraph-powered detective game where you investigate a murder by interviewing suspects, searching for evidence, checking contradictions, and querying timelines. Built with 5 specialized AI agents, vector search, and persistent memory.

## Architecture Overview
The system uses a **graph-based orchestration** where user input is routed to the appropriate agent. Each agent updates a shared state, and the Consistency Validator runs automatically after key actions.

User Input → Router → (Suspect/Evidence/Narrator/Timeline/Consistency) → State Update → Output


### Agents (5 total)
| Agent | Responsibility |
|-------|----------------|
| **Narrator** | Immersive scene descriptions, location updates |
| **Suspect (Butler)** | Dialogue, hidden secret (gambling debt), evolving alibi |
| **Suspect (Lady Evelyn)** | Dialogue, hidden secret (affair/divorce), evolving alibi |
| **Evidence Manager** | Searches Faiss vector DB, adds discovered evidence to state |
| **Consistency Validator** | Detects contradictions between suspect statements, timeline, evidence |

### Tools
- **EvidenceLookupTool**: Uses FAISS + sentence-transformers to retrieve relevant clues from a pre‑indexed corpus.
- **TimelineQueryTool**: Searches the investigation timeline by keyword or time.

### Persistent State
`InvestigationState` (Pydantic model) maintains:
- `discovered_evidence` – clues found
- `suspect_states` – each suspect’s statements, alibi, secret, status
- `timeline` – list of events with timestamps
- `current_location` – dynamic scene location
- `action_history` – log of all detective actions
- `contradictions` – flagged inconsistencies

## Graph Workflow
┌─────────────┐
│ User Input │
└──────┬──────┘
▼
┌─────────────┐
│ Router │ (classify intent)
└──────┬──────┘
│
┌───────────────┼───────────────┬───────────────┐
▼ ▼ ▼ ▼
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ Narrator │ │ Suspect │ │ Evidence │ │Timeline │
└────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘
│ │ │ │
│ └───────┬───────┘ │
│ ▼ │
│ ┌─────────────────┐ │
│ │ Consistency │ ← auto-run │
│ │ Validator │ │
│ └────────┬────────┘ │
│ │ │
└───────────────────────┼───────────────────────┘
▼
┌─────────────┐
│ END / Loop │
└─────────────┘


## Design Decisions
- **Router**: Rule‑based (keyword matching) instead of LLM-based – faster, cheaper, deterministic.
- **Vector DB**: FAISS with `all-MiniLM-L6-v2` – lightweight, fast, works entirely in‑memory.
- **State Management**: Immutable Pydantic models with `.copy(deep=True)` to avoid accidental mutation.
- **Failure Handling**: Every LLM call wrapped in try/except; malformed JSON cleaned; dead suspects blocked.
- **Observability**: Console logs + LangSmith tracing (optional) for full graph and LLM visibility.

## Limitations
- Only two suspects (easily extendable)
- Timeline tool uses substring matching, not semantic search
- State is in‑memory only – no persistent database across sessions
- Gemini free tier has rate limits (20 req/min)

## Future Improvements
- Add more suspects and locations
- Use a vector DB for timeline search (e.g., Qdrant)
- Implement a long‑term memory store (SQLite/Redis)
- Add a “supervisor” agent that suggests next steps
- Support natural language timeline queries via LLM

## Demo Commands to Try
Interview the butler
Search the library for evidence
Where was Lady Evelyn at 9:45 PM?
Check contradictions
Show all evidence collected
Compare timelines
Search the victim's apartment


## Technologies Used
- **LangGraph** – state machine orchestration
- **Gemini 2.5 Flash** – LLM for narration, dialogue, validation
- **FAISS + Sentence‑Transformers** – evidence retrieval
- **Pydantic** – structured state & outputs
- **LangSmith** – tracing & debugging
