sequenceDiagram
    actor User
    participant UI as Frontend (index.html / script.js)
    participant API as FastAPI (app.py)
    participant RAG as RAGSystem (rag_system.py)
    participant Session as SessionManager (session_manager.py)
    participant AI as AIGenerator (ai_generator.py)
    participant Claude as Claude API
    participant Tools as ToolManager (search_tools.py)
    participant VS as VectorStore (vector_store.py)
    participant DB as ChromaDB

    User->>UI: Types question, presses Enter
    UI->>UI: Show loading spinner, disable input
    UI->>API: POST /api/query {query, session_id}

    API->>Session: create_session() if no session_id
    Session-->>API: session_id

    API->>RAG: query(query, session_id)
    RAG->>Session: get_conversation_history(session_id)
    Session-->>RAG: last 2 Q&A exchanges (or null)

    RAG->>AI: generate_response(query, history, tools)
    AI->>Claude: messages.create(system+history, user query, tools)

    alt Claude decides to search
        Claude-->>AI: stop_reason="tool_use"
        AI->>Tools: execute_tool("search_course_content", query, course_name?, lesson_number?)

        opt course_name provided
            Tools->>VS: _resolve_course_name(course_name)
            VS->>DB: query course_catalog (semantic)
            DB-->>VS: exact course title
        end

        VS->>DB: query course_content (semantic + filters)
        DB-->>VS: top 5 matching chunks
        VS-->>Tools: SearchResults
        Tools-->>AI: formatted chunks + stores sources

        AI->>Claude: messages.create(original + tool result)
        Claude-->>AI: final answer text
    else Claude answers directly
        Claude-->>AI: answer text
    end

    AI-->>RAG: response text
    RAG->>Tools: get_last_sources() then reset_sources()
    RAG->>Session: add_exchange(session_id, query, response)
    RAG-->>API: (answer, sources)

    API-->>UI: {answer, sources, session_id}
    UI->>UI: Remove spinner, render Markdown answer
    UI->>UI: Show collapsible sources if any
    UI->>User: Displays response
