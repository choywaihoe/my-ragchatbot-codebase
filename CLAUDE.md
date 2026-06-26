# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Application

```bash
# Install dependencies
uv sync

# Set up environment (copy and fill in ANTHROPIC_API_KEY)
cp .env.example .env

# Start the server (runs from repo root)
./run.sh

# Or manually
cd backend && uv run uvicorn app:app --reload --port 8000
```

The app runs at `http://localhost:8000`. On startup it auto-ingests all `.txt/.pdf/.docx` files from `docs/` into ChromaDB, skipping courses already indexed.

## Architecture

This is a full-stack RAG (Retrieval-Augmented Generation) chatbot. The backend is a FastAPI app in `backend/`; the frontend is static HTML/CSS/JS in `frontend/` served by FastAPI itself.

### Query Flow

1. `POST /api/query` → `app.py` creates a session if needed, delegates to `RAGSystem.query()`
2. `rag_system.py` fetches conversation history, then calls `AIGenerator.generate_response()` with the `search_course_content` tool
3. Claude decides whether to invoke the tool. If it does, `AIGenerator._handle_tool_execution()` runs the search and makes a second Claude call to synthesize the answer
4. `CourseSearchTool` → `VectorStore.search()` optionally resolves a fuzzy course name via semantic search on `course_catalog`, then queries `course_content` with metadata filters
5. Sources and response return to the frontend; the exchange is saved to `SessionManager`

### Document Ingestion Flow

`DocumentProcessor.process_course_document()` parses structured `.txt` files:
- Lines 1–3: `Course Title:`, `Course Link:`, `Course Instructor:`
- Body: `Lesson N: <title>` markers followed by content; optional `Lesson Link:` on the line after each marker

Lesson text is sentence-split into overlapping chunks (800 chars, ~100 char overlap). Two ChromaDB collections are populated:
- `course_catalog` — one entry per course (title, instructor, link, lessons as JSON)
- `course_content` — one entry per chunk with `{course_title, lesson_number, chunk_index}` metadata

### Key Configuration (`backend/config.py`)

| Setting | Value | Effect |
|---|---|---|
| `ANTHROPIC_MODEL` | `claude-sonnet-4-20250514` | Model used for generation |
| `EMBEDDING_MODEL` | `all-MiniLM-L6-v2` | Sentence-transformer for ChromaDB |
| `CHUNK_SIZE` | 800 | Max characters per content chunk |
| `CHUNK_OVERLAP` | 100 | Overlap between chunks |
| `MAX_RESULTS` | 5 | Top-k results from vector search |
| `MAX_HISTORY` | 2 | Q&A exchanges kept per session |
| `CHROMA_PATH` | `./chroma_db` | Persisted ChromaDB location (relative to `backend/`) |

### Adding a New Tool

1. Subclass `Tool` (abstract base in `search_tools.py`) and implement `get_tool_definition()` and `execute()`
2. Register it in `RAGSystem.__init__()` via `self.tool_manager.register_tool(your_tool)`

The `ToolManager` automatically includes registered tools in every Claude call and routes `tool_use` responses to the correct `execute()` method.

### Course Document Format

```
Course Title: My Course
Course Link: https://example.com/course
Course Instructor: Jane Doe

Lesson 0: Introduction
Lesson Link: https://example.com/lesson/0
[lesson content...]

Lesson 1: Core Concepts
[lesson content...]
```
