# Architecture Diagram

## Component Flow
┌─────────────────────────────────────────────────────────────────────────────┐
│ INVESTIGATION STATE │
│ discovered_evidence │ suspect_states │ timeline │ current_location │ etc. │
└─────────────────────────────────────────────────────────────────────────────┘
▲
│ read/write
│
┌─────────────────────────────────────────────────────────────────────────────┐
│ LANGGRAPH ORCHESTRATOR │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │
│ │Narrator │ │ Suspect │ │Evidence │ │Timeline │ │Consistency│ │
│ │ Node │ │ Node │ │ Manager │ │ Tool │ │ Validator │ │
│ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ │
│ │ │ │ │ │ │
│ └────────────┴────────────┴────────────┴────────────┘ │
│ │ │
│ Conditional Edge │
│ (router: narrator/suspect/evidence/timeline) │
└─────────────────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ TOOLS │
│ ┌─────────────────────────┐ ┌─────────────────────────┐ │
│ │ EvidenceLookupTool │ │ TimelineQueryTool │ │
│ │ • FAISS index │ │ • regex search │ │
│ │ • sentence-transformers │ │ • returns events │ │
│ └─────────────────────────┘ └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ EXTERNAL SERVICES │
│ ┌─────────────────┐ ┌─────────────────┐ │
│ │ Gemini LLM │ │ LangSmith │ │
│ │ (narrator, │ │ (tracing/logs) │ │
│ │ suspects, │ │ │ │
│ │ validator) │ │ │ │
│ └─────────────────┘ └─────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘


## State Transitions
| Node | Reads from State | Writes to State |
|------|------------------|------------------|
| Narrator | current_location, discovered_evidence, action_history | last_agent_output, action_history, current_location, graph_step_count |
| Suspect | suspect_states, discovered_evidence | suspect_states (statements/alibi), last_agent_output, action_history |
| Evidence Manager | discovered_evidence | discovered_evidence (adds new), last_agent_output |
| Timeline Tool | timeline | last_agent_output |
| Consistency Validator | suspect_states, timeline, discovered_evidence | contradictions, last_agent_output |
