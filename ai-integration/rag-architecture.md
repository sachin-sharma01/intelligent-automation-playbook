# 🧠 RAG Architecture — Retrieval Augmented Generation

RAG is how you give an LLM access to YOUR data — without hallucination.

---

## The Problem RAG Solves

LLMs know general knowledge up to their training cutoff.  
They don't know:
- Your internal documents
- Your company's regulations
- Your product specs
- Anything recent or private

**RAG fixes this** by retrieving relevant context at query time and feeding it to the LLM.

---

## Two Phases

### Phase 1: Indexing (one-time setup)

```
Your documents (PDF, text, web)
    → Chunking (split into pieces ~500 tokens)
        → Embedding (convert text → vector numbers)
            → Store in Vector DB (Pinecone, pgvector, Chroma)
```

### Phase 2: Query time (every request)

```
User question
    → Embed the question (same embedding model)
        → Similarity search in Vector DB
            → Retrieve top-K relevant chunks
                → Build prompt: question + chunks
                    → Send to LLM
                        → Answer grounded in YOUR data
```

---

## What is an Embedding?

A vector is a list of numbers that represents the *meaning* of text:

```
"GDPR data retention" → [0.23, -0.81, 0.44, 0.12, ...]
                         (1536 numbers for OpenAI embeddings)
```

Similar meaning = similar numbers = close together in vector space.  
This is how semantic search works — finds meaning, not just keywords.

---

## Stack Options

| Component | Options |
|---|---|
| **Embedding model** | OpenAI `text-embedding-3-small`, Cohere, local models |
| **Vector DB** | Supabase (pgvector), Pinecone, Chroma, Weaviate |
| **LLM** | GPT-4, Claude, Mistral, local Llama |
| **Orchestration** | n8n, LangChain, LlamaIndex, custom |

---

## Supabase + pgvector Setup

```sql
-- Enable pgvector extension
create extension if not exists vector;

-- Create table with embedding column
create table documents (
  id text primary key,
  content text,
  embedding vector(1536),  -- 1536 for OpenAI embeddings
  metadata jsonb
);

-- Create index for fast similarity search
create index on documents 
using ivfflat (embedding vector_cosine_ops);
```

### Similarity search query:
```sql
select content, metadata
from documents
order by embedding <-> '[0.23, -0.81, 0.44...]'  -- your query vector
limit 5;
```

`<->` = cosine distance operator — finds closest vectors.

---

## RAG vs Simple Document Upload

| | Simple upload (e.g. Claude.ai) | RAG |
|---|---|---|
| Storage | Context window (temporary) | Vector DB (permanent) |
| Scale | Limited by context size | Millions of documents |
| Retrieval | Full doc loaded | Only relevant chunks |
| Memory | Gone after chat | Persists forever |
| Hallucination risk | Higher | Lower (grounded in data) |

**Simple upload = giving LLM a cheat sheet.**  
**RAG = giving LLM a library with a smart search engine.**

---

## Real Implementation — Regulation Intelligence Pipeline

A personal project that monitors banking regulation newsletters automatically:

```
Scheduled trigger (n8n)
    → Download regulation newsletter (PDF)
        → Extract text
            → Send to Claude API with domain-expert system prompt
                → Claude returns structured JSON:
                    {
                      title, source, category,
                      impact_level, affected_departments,
                      required_actions, deadline, summary
                    }
                → Filter: high/medium impact only
                    → Save to Supabase (structured table)
                        → Notify compliance team via Teams
```

**Next iteration:** Add embedding column to Supabase → enable semantic search on the UI → users can query regulations in natural language.

---

## Prompt Engineering for RAG

System prompt structure that works well:

```
You are a [domain expert] at a [specific organization type].

Your task is to [specific goal].

The organization does the following:
- [activity 1]
- [activity 2]

For each item, return a JSON object with these fields:
- field1: description
- field2: must be one of [value1, value2, value3]
- field3: true/false

Return ONLY valid JSON. No preamble, no explanation.
```

**Key principles:**
- Be specific about the domain
- Constrain output format strictly
- Use enum values where possible — prevents hallucination
- One task per prompt — don't ask for too much at once
