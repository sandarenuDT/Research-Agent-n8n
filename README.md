# AI Research Agent (n8n)

An autonomous AI agent built in n8n that answers questions by checking an internal knowledge base first, falling back to live web search, and (in progress) semantic search over a PDF document — then delivers structured answers via email.

## What it does

- Accepts a question through a chat interface
- Decides which source to check: an internal Google Sheets knowledge base, a live web search, or a PDF knowledge base (via vector search)
- Remembers prior turns in the same conversation
- Returns a structured answer noting which source it used and how confident it is
- Emails the final answer automatically

## Architecture

```
Chat Trigger → AI Agent → Send Email
                  │
        ┌─────────┼─────────┐
    Chat Model   Memory    Tools
                              │
                ┌─────────────┼─────────────┐
          Google Sheets   Web Search   PDF Vector Store
          (knowledge base) (Tavily)    (semantic search)
```

1. **Chat Trigger** — starts the workflow when a user sends a message
2. **AI Agent** — the orchestrator; decides which tool(s) to call and synthesizes the final answer
3. **Chat Model** — Google Gemini, powers the agent's reasoning
4. **Memory** — a windowed buffer that keeps conversation context across turns
5. **Tools**
   - **Google Sheets** — internal knowledge base (company policies, FAQs)
   - **Web Search (Tavily)** — live information for anything not in the knowledge base
   - **PDF Vector Store** — semantic search over an uploaded PDF, using embeddings rather than exact keyword matching
6. **Send Email (Gmail)** — delivers the agent's structured answer

## Tech stack

- [n8n](https://n8n.io) — workflow orchestration (Community edition, free)
- [Google Gemini](https://ai.google.dev) — LLM for agent reasoning
- [Tavily](https://tavily.com) — web search API
- Google Sheets — lightweight structured knowledge base
- Gmail — automated delivery

## Key features

- **RAG (retrieval-augmented generation)** — the agent grounds its answers in real data instead of relying purely on model memory, checking a structured knowledge base and (with the PDF pipeline) a vector-embedded document store
- **Tool orchestration** — the agent autonomously chooses between multiple tools based on the question, rather than following a fixed script
- **Conversational memory** — follow-up questions are understood in context without the user repeating themselves
- **Structured output** — responses follow a consistent JSON schema (answer, source, confidence) for reliable downstream use
- **Automated delivery** — final answers are emailed without manual intervention

## Setup

1. Install n8n (Docker or n8n Cloud free tier)
2. Import `research-agent-workflow.json` into n8n
3. Add credentials:
   - Google Gemini API key
   - Tavily API key
   - Google Sheets OAuth (for your knowledge base sheet)
   - Gmail OAuth (for sending emails)
4. Create a Google Sheet with `Topic` and `Content` columns and populate it with reference data
5. Update the Google Sheets node to point at your sheet
6. Open the built-in chat panel in n8n and start asking questions

## Example

**Input:** "What's the return policy?"
**Agent behavior:** checks the knowledge base tool first, finds a match, returns the answer, marks source as `knowledge_base`, emails the result.

**Input:** "What's the latest news on AI regulation?"
**Agent behavior:** knowledge base has no match, falls back to web search, synthesizes an answer from multiple sources, marks source as `web_search`.

## Roadmap / possible extensions

- Semantic search over PDFs via a vector store (Pinecone/Supabase) — in progress
- Query planning: breaking complex questions into sub-queries before searching
- Source validation and fact cross-referencing across multiple search results
- Markdown-formatted research reports instead of short JSON answers
- Multi-agent routing (a "gatekeeper" agent that delegates to specialized sub-agents)
- Usage logging (every question, answer, and source logged to a sheet for analytics)

## Notes

Built as a learning project to explore agentic workflows, RAG, and tool orchestration using n8n's free Community edition.
