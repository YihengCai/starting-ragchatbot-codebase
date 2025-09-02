# RAG System Query Flow Diagram

```
┌─────────────────┐    POST /api/query     ┌─────────────────┐
│   Frontend      │ ──────────────────────► │   FastAPI       │
│   (script.js)   │                         │   (app.py)      │
│                 │                         │                 │
│ 1. User Input   │                         │ 2. HTTP Handler │
│ 2. sendMessage()│                         │ 3. Create Session│
│ 3. Loading UI   │                         │ 4. Call RAG     │
└─────────────────┘                         └─────────┬───────┘
         ▲                                            │
         │                                            ▼
         │                                  ┌─────────────────┐
         │                                  │   RAG System    │
         │                                  │   (rag_system)  │
         │                                  │                 │
         │                                  │ 5. Format Query │
         │                                  │ 6. Get History  │
         │                                  │ 7. Call AI Gen  │
         │                                  └─────────┬───────┘
         │                                            │
         │                                            ▼
┌─────────────────┐                         ┌─────────────────┐
│   Response      │                         │   AI Generator  │
│   Display       │                         │   (Claude API)  │
│                 │                         │                 │
│ 12. Remove      │                         │ 8. System Prompt│
│     Loading     │                         │ 9. Decide Tools │
│ 13. Render      │                         │ 10. Tool Call?  │
│     Markdown    │                         └─────────┬───────┘
│ 14. Show Sources│                                   │
└─────────────────┘                                   ▼
         ▲                                  ┌─────────────────┐
         │                                  │  Search Tool    │
         │                                  │  (if needed)    │
         │                                  │                 │
         │                                  │ 11a. Vector     │
         │                                  │      Search     │
         │                                  │ 11b. ChromaDB   │
         │                                  │ 11c. Return     │
         │                                  │      Results    │
         │                                  └─────────┬───────┘
         │                                            │
         │          JSON Response                     │
         └────────────────────────────────────────────┘
                    {answer, sources, session_id}
```

## Flow Details

### Phase 1: Frontend Initiation
- User types query in chat input
- `sendMessage()` disables UI, shows loading
- POST request to `/api/query` with query + session_id

### Phase 2: API Gateway
- FastAPI receives request at `/api/query` endpoint
- Creates new session if needed
- Validates request and calls RAG system

### Phase 3: RAG Orchestration
- Formats query with instructional prompt
- Retrieves conversation history from session manager
- Passes query to AI generator with available tools

### Phase 4: AI Processing
- Claude receives query with system prompt
- Decides whether to use search tool based on query type
- If course-specific: triggers search tool
- If general knowledge: answers directly

### Phase 5: Vector Search (Conditional)
- CourseSearchTool performs semantic search
- ChromaDB returns relevant course chunks
- Results passed back to Claude for synthesis

### Phase 6: Response Assembly
- Claude generates final answer using search results
- Sources extracted from tool usage
- Response formatted as JSON

### Phase 7: Frontend Rendering
- Receives JSON response
- Removes loading animation
- Renders answer with markdown formatting
- Shows collapsible sources section

## Key Components

- **Session Management**: Maintains conversation context across requests
- **Tool Selection**: AI autonomously decides when to search vs use general knowledge
- **Vector Database**: ChromaDB with sentence transformer embeddings
- **Error Handling**: Graceful fallbacks at each level
- **Real-time UI**: Progressive loading and response rendering