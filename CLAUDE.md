# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Course Materials RAG (Retrieval-Augmented Generation) System** - a full-stack web application that enables users to query course materials and receive intelligent, context-aware responses using semantic search and AI-powered generation.

## Architecture

### Core RAG Pipeline
The system follows a modular RAG architecture with clear separation of concerns:

1. **Document Processing** (`document_processor.py`) - Handles course file parsing, metadata extraction, and intelligent text chunking with sentence-based boundaries and configurable overlap
2. **Vector Storage** (`vector_store.py`) - ChromaDB integration with SentenceTransformer embeddings for semantic similarity search
3. **RAG Orchestration** (`rag_system.py`) - Main coordinator that connects document processing, vector search, AI generation, and session management
4. **AI Generation** (`ai_generator.py`) - Anthropic Claude integration with tool-based search capabilities and system prompts for educational responses
5. **Tool System** (`search_tools.py`) - Extensible tool architecture with CourseSearchTool for semantic course content retrieval

### Session & Context Management
- **Session Manager** (`session_manager.py`) - Maintains conversation history with configurable limits
- **Models** (`models.py`) - Pydantic data structures for Course, Lesson, and CourseChunk entities
- **Configuration** (`config.py`) - Centralized settings for chunk sizes, API keys, embedding models, and database paths

### API Layer
- **FastAPI Application** (`app.py`) - REST endpoints with CORS support, static file serving, and automatic document loading
- **Frontend** (`frontend/`) - Vanilla HTML/CSS/JavaScript with markdown rendering and real-time chat interface

## Development Commands

### Environment Setup
```bash
# Install dependencies
uv sync

# Set up environment variables (copy .env.example to .env and add your Anthropic API key)
cp .env.example .env
```

### Running the Application
```bash
# Quick start using shell script
chmod +x run.sh
./run.sh

# Manual start
cd backend
uv run uvicorn app:app --reload --port 8000
```

**Important**: Always use `uv` for all dependency management and Python execution:
- Use `uv sync` to install dependencies (not `pip install`)  
- Use `uv add [package]` to add new dependencies (not `pip install [package]`)
- Use `uv run [command]` for all Python execution (not direct python commands)
- Never use `pip` directly - this project uses `uv` for package management

### Access Points
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs

## Document Format

Course documents in `docs/` follow this structured format:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: Introduction
Lesson Link: [lesson_url]
[lesson content]

Lesson 1: Next Topic
[lesson content]
```

## Key Components Integration

### Query Flow
1. Frontend (`script.js`) sends POST to `/api/query`
2. FastAPI (`app.py`) creates session and calls RAG system
3. RAG system (`rag_system.py`) formats query and retrieves conversation history
4. AI Generator (`ai_generator.py`) processes with Claude, optionally using search tools
5. Course Search Tool performs vector similarity search when needed
6. Response assembled with sources and returned through the chain

### Tool System Architecture
The tool system is designed for extensibility:
- `Tool` abstract base class defines interface for all tools
- `ToolManager` handles tool registration and execution
- `CourseSearchTool` implements semantic search with course/lesson filtering
- AI decides autonomously when to use tools based on query type

### Configuration Dependencies
- Requires Anthropic API key in `.env`
- ChromaDB storage path configurable in `config.py`
- Embedding model: `all-MiniLM-L6-v2` (SentenceTransformers)
- Claude model: `claude-sonnet-4-20250514`
- Text chunking: 800 characters with 100 character overlap

## Database & Storage

- **Vector Database**: ChromaDB with persistent storage in `./chroma_db`
- **Embeddings**: Generated using SentenceTransformers on document chunks
- **Metadata**: Course title, lesson number, and chunk index stored with each vector
- **Session Storage**: In-memory session management with conversation history

## Frontend Integration

- **Markdown Rendering**: Uses marked.js for Claude's markdown responses  
- **Real-time UI**: Loading animations and progressive response rendering
- **Source Attribution**: Collapsible sources section showing course/lesson references
- **Session Persistence**: Maintains conversation context across queries