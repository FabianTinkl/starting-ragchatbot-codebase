# RAG System Request Flow Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend<br/>(script.js)
    participant API as FastAPI<br/>(app.py)
    participant RAG as RAG System<br/>(rag_system.py)
    participant SM as Session Manager
    participant AI as AI Generator<br/>(ai_generator.py)
    participant Claude as Anthropic Claude
    participant TM as Tool Manager<br/>(search_tools.py)
    participant VS as Vector Store<br/>(ChromaDB)

    U->>FE: Types query & clicks send
    FE->>FE: Disable input, show loading
    FE->>API: POST /api/query<br/>{query, session_id}
    
    API->>RAG: rag_system.query(query, session_id)
    
    RAG->>SM: get_conversation_history(session_id)
    SM-->>RAG: Previous messages
    
    RAG->>AI: generate_response(query, history, tools)
    
    AI->>Claude: Initial API call<br/>System prompt + query + tools
    
    alt Claude decides to use search tool
        Claude-->>AI: Tool use request<br/>(search_course_content)
        AI->>TM: execute_tool(name, params)
        TM->>VS: search_content(query, filters)
        VS-->>TM: Relevant chunks + metadata
        TM-->>AI: Search results
        TM->>TM: Track sources for citation
        
        AI->>Claude: Follow-up call with<br/>search results
        Claude-->>AI: Final synthesized response
    else Claude uses general knowledge
        Claude-->>AI: Direct response
    end
    
    AI-->>RAG: Generated response text
    RAG->>TM: get_last_sources()
    TM-->>RAG: Source citations
    RAG->>SM: add_exchange(session_id, query, response)
    RAG-->>API: (response, sources)
    
    API-->>FE: JSON response<br/>{answer, sources, session_id}
    
    FE->>FE: Remove loading, display response
    FE->>FE: Show sources, re-enable input
    FE-->>U: Formatted response with sources

    Note over U,VS: Document Processing (Startup)
    Note over VS: 4 course files → parsed → chunked → embedded → stored
```

## Component Responsibilities

### Frontend Layer
- **User Interface**: Input handling, loading states, message display
- **Session Management**: Tracks session ID across requests
- **Error Handling**: Network failures, API errors

### API Layer  
- **Request Routing**: FastAPI endpoints (`/api/query`, `/api/courses`)
- **Response Formatting**: Structured JSON responses
- **Error Handling**: HTTP status codes, exception handling

### RAG Orchestration
- **Component Coordination**: Manages all system components
- **Query Processing**: Formats prompts for AI
- **Response Assembly**: Combines AI output with sources

### AI Generation
- **Claude Integration**: Anthropic API calls with tools
- **Tool Orchestration**: Manages tool execution flow
- **Context Management**: Handles conversation history

### Search & Storage
- **Semantic Search**: Vector similarity matching
- **Source Tracking**: Citation management
- **Data Persistence**: ChromaDB vector storage

### Session Management
- **Conversation History**: Multi-turn dialogue support
- **Context Preservation**: Maintains conversational flow

## Key Data Transformations

1. **User Query** → API Request JSON
2. **Text Documents** → Chunked + Embedded Vectors  
3. **Search Query** → Vector Similarity Results
4. **Raw Results** → Contextualized Prompt
5. **AI Response** → Formatted Frontend Display