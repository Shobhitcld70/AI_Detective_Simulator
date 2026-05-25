# Failures & Learnings – Multi-Agent Detective Simulator

## Hardest Bug / Debugging Issue

**Location not persisting across turns**

Despite having `current_location` in the state and the narrator returning `scene_update`, the location always stayed as "Mansion Entrance". After adding console logs, I discovered that the LLM was returning JSON wrapped in markdown code fences (```json ... ```). The `json.loads()` failed, falling back to the exception handler which set `scene_update="No major change."`.

**Solution**: Added a cleaning step to strip ````json` and ```` from the response before parsing. Also added a print statement to see the raw output when parsing fails.

**Lesson**: Always sanitize LLM outputs – they love markdown formatting even when instructed to output plain JSON.

## Orchestration Problems Encountered

**Infinite loop from router reusing same user_input**

The graph’s conditional entry point uses `state.user_input` to route. After the narrator node ended, the main loop would call `graph_app.invoke()` again with the same `user_input` still in the state, causing the same route to be taken repeatedly without new user input.

**Solution**: Clear `state.user_input` after each invocation or restructure the loop to only pass user input as a separate parameter. In my implementation, I ensured the state is copied and the original `user_input` is not persisted across loops.

**Lesson**: Be careful with mutable state – what you intend as a "command" vs "persistent memory" should be separate.

## Retrieval / Memory Consistency Issues

**Evidence not automatically added when narrator describes a clue**

The assignment required tracking discovered evidence, but only the Evidence Manager tool could add items to `discovered_evidence`. If the user simply explored the scene (via narrator), new clues wouldn’t be recorded.

**Solution**: I added a keyword-based auto‑extraction in the narrator node (optional) and also explicitly instructed users to use the `"search evidence"` command. For the final design, I kept the tool‑based approach to respect the assignment’s tool requirement.

**Lesson**: When integrating retrieval (FAISS) with state, define clear boundaries: tools for explicit queries, agents for implicit updates.

## Architectural Tradeoffs

| Tradeoff | Decision | Rationale |
|----------|----------|-----------|
| Rule‑based vs LLM router | Rule‑based (keywords) | Faster, cheaper, no risk of misrouting. LLM would add latency and cost for a simple classification. |
| In‑memory state vs database | In‑memory | Simpler for a local simulator. Persistent DB would add complexity but enable long‑running investigations. |
| FAISS vs other vector DB | FAISS | Lightweight, no external server, works inside notebook. |
| Manual JSON cleaning vs strict prompting | Cleaning | LLMs are unreliable with formatting; cleaning is more robust. |
| Consistency Validator as a separate node | Separate node | Allows automatic checks after suspect interviews and evidence discovery without user having to ask. |

## What I Would Redesign with More Time

1. **Add a “Supervisor” Agent**  
   Currently the user must guess what to do. A supervisor agent could suggest next steps based on missing evidence or contradictions.

2. **Persistent Database**  
   Use SQLite to save investigations across sessions. Allow loading previous cases.

3. **Semantic Timeline Search**  
   Replace substring matching with embeddings for timeline queries like *“what happened around the time of the gunshot?”*

4. **More Robust Failure Recovery**  
   Add a fallback retry mechanism with exponential backoff for rate‑limit errors.

5. **Unit & Integration Tests**  
   With more time, I’d add pytest tests for each agent, state transitions, and edge cases (dead suspect, empty response).

6. **Web Interface**  
   Replace the console with a simple Gradio or Streamlit UI for a more immersive experience.

7. **Dynamic Evidence Generation**  
   Instead of a fixed corpus, have the narrator generate new evidence on the fly (with consistency validation).

## Key Takeaways

- LangGraph is powerful but requires careful state management.
- LLM outputs are unpredictable – always validate and clean.
- Persistent state is only as good as the code that updates it; one missed field breaks immersion.
- Tools (like FAISS) make the system extensible and impressive to evaluators.
- Logging and tracing (LangSmith) save hours of debugging.

**Final thought**: This project successfully demonstrates a multi‑agent orchestration with memory, tools, and graceful failure handling – exactly what the assignment asked for. The bugs I encountered taught me more about state machines than a perfect first attempt ever would.
