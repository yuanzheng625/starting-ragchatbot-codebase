# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

This is a Retrieval-Augmented Generation (RAG) system for querying course materials. The system uses semantic search with ChromaDB and Anthropic's Claude AI to provide intelligent responses about educational content.

### Core Components

- **FastAPI Backend** (`backend/app.py`): Main web server serving both API endpoints and static frontend files
- **RAG System** (`backend/rag_system.py`): Central orchestrator coordinating all components
- **Vector Store** (`backend/vector_store.py`): ChromaDB-based semantic search with two collections:
  - `course_catalog`: Course metadata for name resolution
  - `course_content`: Chunked course content for semantic search
- **Document Processor** (`backend/document_processor.py`): Processes course documents with specific format expectations
- **AI Generator** (`backend/ai_generator.py`): Claude API wrapper with tool integration
- **Search Tools** (`backend/search_tools.py`): Tool-based search interface for AI model
- **Session Manager** (`backend/session_manager.py`): Conversation history management

### Document Format

Course documents follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: Introduction
Lesson Link: [url]
[content]

Lesson 1: [title]
[content]
```

## Common Commands

### Development
```bash
# Install dependencies
uv sync

# Start development server
./run.sh
# OR manually:
cd backend && uv run uvicorn app:app --reload --port 8000

# Access application
# Web Interface: http://localhost:8000
# API Documentation: http://localhost:8000/docs
```

### Configuration
- Create `.env` file with: `ANTHROPIC_API_KEY=your_key_here`
- Documents go in `docs/` directory (loaded automatically on startup)

### Architecture Notes

- **Two-stage vector search**: Course name resolution via `course_catalog`, then content search in `course_content`
- **Tool-based AI interaction**: AI model uses search tools rather than direct vector retrieval
- **Chunking strategy**: Sentence-based chunks with configurable overlap (800 chars, 100 overlap)
- **Session management**: Conversation history tracked by session ID
- **Static file serving**: Frontend served directly from FastAPI with development cache headers
- **Persistent storage**: ChromaDB database persisted in `backend/chroma_db/`

## Key Configuration Values

All configurable in `backend/config.py`:
- `CHUNK_SIZE`: 800 characters
- `CHUNK_OVERLAP`: 100 characters  
- `MAX_RESULTS`: 5 search results
- `MAX_HISTORY`: 2 conversation turns
- `EMBEDDING_MODEL`: "all-MiniLM-L6-v2"
- `ANTHROPIC_MODEL`: "claude-sonnet-4-20250514"