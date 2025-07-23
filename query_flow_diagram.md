# Query Processing Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                           FRONTEND (script.js)                      │
├─────────────────────────────────────────────────────────────────────┤
│  User Input → sendMessage() → Disable UI → Show Loading             │
│                                    │                                 │
│                                    ▼                                 │
│              POST /api/query {query, session_id}                     │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        FASTAPI (app.py)                             │
├─────────────────────────────────────────────────────────────────────┤
│  @app.post("/api/query") → QueryRequest → Create/Get Session        │
│                                    │                                 │
│                                    ▼                                 │
│                        rag_system.query()                           │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      RAG SYSTEM (rag_system.py)                     │
├─────────────────────────────────────────────────────────────────────┤
│  Format Prompt → Get History → ai_generator.generate_response()     │
│  "Answer this question about course materials: {query}"             │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    AI GENERATOR (ai_generator.py)                    │
├─────────────────────────────────────────────────────────────────────┤
│  Build System Prompt + History → Prepare API Call                   │
│                                    │                                 │
│                                    ▼                                 │
│            Claude API Call (with tools=search_tool)                 │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      CLAUDE AI DECISION                             │
├─────────────────────────────────────────────────────────────────────┤
│  Does query need course search? → tool_choice: "auto"               │
│                                    │                                 │
│           ┌───────────────────────┼───────────────────────┐         │
│           ▼                       ▼                       ▼         │
│    Use Search Tool          Direct Answer           No Tools        │
└───────────┬───────────────────────────────────────────────┬─────────┘
            │                                               │
            ▼                                               │
┌─────────────────────────────────────────────────────────────────────┐
│                  SEARCH TOOL (search_tools.py)                      │
├─────────────────────────────────────────────────────────────────────┤
│  Parse Query → vector_store.search() → Return Results               │
│                                    │                                 │
│                                    ▼                                 │
│                        VECTOR STORE                                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              CHROMADB (vector_store.py)                     │    │
│  │  ┌─────────────────┐  ┌─────────────────────────────────┐   │    │
│  │  │ course_catalog  │  │      course_content             │   │    │
│  │  │ (metadata)      │  │      (text chunks)              │   │    │
│  │  └─────────────────┘  └─────────────────────────────────┘   │    │
│  │            │                        │                      │    │
│  │            ▼                        ▼                      │    │
│  │    Resolve Course Name    →    Semantic Search             │    │
│  │                                     │                      │    │
│  │                                     ▼                      │    │
│  │                           Return Relevant Chunks           │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        CLAUDE SYNTHESIS                             │
├─────────────────────────────────────────────────────────────────────┤
│  Search Results + Query + History → Generate Final Answer           │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    RESPONSE ASSEMBLY                                │
├─────────────────────────────────────────────────────────────────────┤
│  QueryResponse{answer, sources, session_id} → JSON                  │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      FRONTEND DISPLAY                               │
├─────────────────────────────────────────────────────────────────────┤
│  Remove Loading → addMessage(response) → Enable UI → Ready          │
└─────────────────────────────────────────────────────────────────────┘

## Key Components:

**Session Management**: Tracks conversation history per user session
**Tool-Based RAG**: Claude autonomously decides when to search course content  
**Vector Search**: ChromaDB with SentenceTransformer embeddings
**Dual Collections**: 
  - `course_catalog`: Course metadata for name resolution
  - `course_content`: Text chunks for semantic search