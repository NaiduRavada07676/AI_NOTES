# Session 4 — RAG: Retrieval-Augmented Generation
## 1. Why Do We Even Need RAG?

### The Core Problem with LLMs

LLMs (like GPT, Claude, etc.) have a **knowledge cutoff** — they only know what they were trained on. After that date, they're clueless.

**Think of it like this:**

> You studied for your exam 6 months ago. Someone asks you about a news article from last week. You have no idea — it wasn't in your study material.

### The 3 Big LLM Limitations

| #   | Limitation                 | Example                                                                       |
| --- | -------------------------- | ----------------------------------------------------------------------------- |
| 1   | **Knowledge cutoff**       | "What's the latest Express.js version?" — LLM might say 4.x when 5.x is out   |
| 2   | **No access to YOUR data** | "What's our company's refund policy?" — LLM has never seen your internal docs |
| 3   | **Hallucination**          | When LLM doesn't know, it confidently makes stuff up                          |

### The Solution: Give LLM a Cheat Sheet

RAG = **Before asking the LLM a question, first search your own data, find relevant info, and paste it into the prompt.**

It's like an open-book exam — the LLM gets to "look up" relevant documents before answering.

```
┌─────────────────────────────────────────────────────┐
│                WITHOUT RAG                          │
│                                                     │
│  User Question ──────────► LLM ──────► Answer       │
│                           (guesses)   (may be wrong)│
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│                WITH RAG                             │
│                                                     │
│  User Question ──► Search Your Docs ──► LLM ──► Answer │
│                    (find relevant info)  (informed)  (accurate) │
└─────────────────────────────────────────────────────┘
```

### Real-World Analogy

| Scenario               | Without RAG               | With RAG                                  |
| ---------------------- | ------------------------- | ----------------------------------------- |
| Student answering exam | Closed book (memory only) | Open book (can look up notes)             |
| Doctor diagnosing      | From memory only          | Checks patient's medical records first    |
| Customer support bot   | Generic answers           | Reads YOUR company's knowledge base first |

---

## 2. What is RAG? (The Full Picture)

### RAG = Retrieval-Augmented Generation

Break it down:

- **Retrieval** → Search and find relevant documents/chunks
- **Augmented** → Add those chunks to the LLM's prompt (augment = enhance)
- **Generation** → LLM generates an answer using that context

### The RAG Pipeline — Visual Diagram

```
┌──────────────────── RAG PIPELINE ────────────────────────┐
│                                                          │
│  PHASE 1: INDEXING (one-time setup)                      │
│  ┌─────────┐    ┌─────────┐    ┌──────────┐             │
│  │  Your   │───►│  Chunk  │───►│ Embed &  │             │
│  │  Docs   │    │  (split)│    │  Store   │             │
│  └─────────┘    └─────────┘    └──────────┘             │
│   PDFs, code,    Split into     Convert to               │
│   wikis, DBs     small pieces   vectors → Vector DB      │
│                                                          │
│  PHASE 2: QUERYING (every user question)                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌──────┐ │
│  │  User   │───►│ Embed   │───►│ Search  │───►│  LLM │ │
│  │ Question│    │ Query   │    │ VectorDB│    │      │ │
│  └─────────┘    └─────────┘    └─────────┘    └──────┘ │
│                  Convert Q      Find top-K     Generate  │
│                  to vector      similar chunks  answer   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Step-by-Step Breakdown

**Phase 1: Indexing (Preparation — done once)**

```
Step 1: Collect documents (PDFs, code, wikis, CSVs, etc.)
Step 2: Chunk them (split into small pieces, ~500-1000 tokens each)
Step 3: Embed each chunk (convert text → vector using embedding model)
Step 4: Store vectors in a Vector Database (Pinecone, ChromaDB, Weaviate)
```

**Phase 2: Querying (Runtime — every question)**

```
Step 1: User asks a question
Step 2: Embed the question (same embedding model)
Step 3: Search Vector DB for top-K most similar chunks
Step 4: Stuff those chunks into the LLM prompt as "context"
Step 5: LLM generates answer grounded in your actual data
```

### Student-Friendly Example

Imagine you're building a chatbot for your college:

```
Documents: Course syllabus, exam schedule, faculty list, hostel rules

Student asks: "When is the AI exam?"

WITHOUT RAG:
  LLM: "I don't have access to your college schedule." (or worse, halluccinates a date)

WITH RAG:
  1. Embed the question → vector
  2. Search vector DB → finds chunk: "AI Mid-Sem: March 15, End-Sem: May 20"
  3. Prompt to LLM: "Based on this: [AI Mid-Sem: March 15, End-Sem: May 20]
                     Answer: When is the AI exam?"
  4. LLM: "The AI mid-semester exam is on March 15 and the end-semester is May 20."
```

---

## 3. Types of RAG Pipelines

### Type 1: Naive RAG (Basic)

The simplest form. Just retrieve and generate.

```
Question → Embed → Search → Top-K chunks → Stuff into prompt → LLM → Answer
```

**Pros:** Simple, fast to build
**Cons:** Retrieved chunks may be irrelevant, no re-ranking, can hit context limits

---

### Type 2: Advanced RAG

Adds pre-retrieval and post-retrieval optimization.

```
Question → Query Rewriting → Embed → Search → Re-Rank → Filter → LLM → Answer
```

**Extra steps:**

- **Query rewriting** — rephrase question for better search results
- **Re-ranking** — score retrieved chunks by relevance, keep only the best
- **Filtering** — remove duplicates, outdated info

**Example:**

```
User asks: "How do I fix the login bug?"

Naive RAG might retrieve:
  - Chunk about login UI styling (irrelevant)
  - Chunk about auth middleware (relevant)
  - Chunk about database setup (somewhat relevant)

Advanced RAG re-ranks:
  1. Auth middleware chunk (score: 0.95) ← keeps this
  2. Database setup (score: 0.72) ← keeps this
  3. Login UI styling (score: 0.3) ← discards this
```

---

### Type 3: Modular RAG

Mix and match components like LEGO blocks.

```
┌──────────────────────────────────────────────────┐
│              MODULAR RAG                          │
│                                                  │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌───────┐ │
│  │ Router │─►│Retrieve│─►│Re-Rank │─►│ LLM   │ │
│  └────────┘  └────────┘  └────────┘  └───────┘ │
│       │           │                              │
│       ▼           ▼                              │
│  Decides      Multiple                           │
│  which DB     sources:                           │
│  to search    - Vector DB                        │
│               - SQL DB                           │
│               - Web Search                       │
│               - API calls                        │
└──────────────────────────────────────────────────┘
```

**Use case:** Enterprise apps where data lives in multiple systems (docs in Notion, code in GitHub, data in PostgreSQL).

---

### Type 4: Agentic RAG

The LLM itself decides _what_ to retrieve and _when_. It can:

- Decide if it needs to search at all
- Choose which tool/database to query
- Make multiple searches iteratively
- Self-correct if first retrieval wasn't good enough

```
User Question
     │
     ▼
┌─────────┐    "Do I need more info?"
│  Agent  │◄──────────────────────────┐
│  (LLM)  │                           │
└────┬────┘                           │
     │ Yes → Search                   │
     ▼                                │
┌─────────┐    Got results            │
│  Tool   │────────────────────────────┘
│ (Search)│
└─────────┘
     │ No more info needed
     ▼
  Final Answer
```

---

### Comparison Table

| Type         | Complexity | Best For                           | Example                        |
| ------------ | ---------- | ---------------------------------- | ------------------------------ |
| Naive RAG    | Low        | Simple Q&A bots, prototypes        | FAQ chatbot                    |
| Advanced RAG | Medium     | Production apps needing accuracy   | Customer support               |
| Modular RAG  | High       | Multi-source enterprise systems    | Internal knowledge platform    |
| Agentic RAG  | Highest    | Complex research, multi-step tasks | Code assistant, research agent |

---

## 4. RAG Bottlenecks & Common Problems

### The 6 Bottlenecks

```
┌──────────────────────────────────────────────────────────┐
│                  RAG BOTTLENECK MAP                       │
│                                                          │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐         │
│  │ CHUNKING │────►│EMBEDDING │────►│ RETRIEVAL│         │
│  │ Too big? │     │ Wrong    │     │ Wrong    │         │
│  │ Too small│     │ model?   │     │ results? │         │
│  └──────────┘     └──────────┘     └──────────┘         │
│       ▼                                  ▼               │
│  ┌──────────┐                      ┌──────────┐         │
│  │ CONTEXT  │                      │GENERATION│         │
│  │ Window   │                      │ LLM still│         │
│  │ Overflow │                      │ wrong?   │         │
│  └──────────┘                      └──────────┘         │
└──────────────────────────────────────────────────────────┘
```

| #   | Bottleneck                  | Problem                                                  | Fix                                                            |
| --- | --------------------------- | -------------------------------------------------------- | -------------------------------------------------------------- |
| 1   | **Bad chunking**            | Chunks too big = noise. Too small = missing context.     | Use semantic chunking (split by meaning, not just token count) |
| 2   | **Wrong embedding model**   | Generic embeddings don't capture domain-specific meaning | Use domain-specific embeddings or fine-tune them               |
| 3   | **Irrelevant retrieval**    | Top-K returns wrong chunks                               | Add re-ranking, hybrid search (keyword + semantic)             |
| 4   | **Context window overflow** | Too many chunks exceed LLM's token limit                 | Compress chunks, use map-reduce, increase K only when needed   |
| 5   | **Lost in the middle**      | LLM ignores info in the middle of long contexts          | Put most relevant chunks first and last                        |
| 6   | **Stale data**              | Documents changed but vectors weren't updated            | Schedule re-indexing, track document versions                  |

### Student Analogy

Think of RAG like a library system:

| Bottleneck           | Library Analogy                                                                    |
| -------------------- | ---------------------------------------------------------------------------------- |
| Bad chunking         | Books are either 1-page pamphlets (too small) or 1000-page encyclopedias (too big) |
| Wrong embeddings     | The librarian files cooking books under "engineering"                              |
| Irrelevant retrieval | You ask for "Python" and get books about snakes                                    |
| Context overflow     | You bring 50 books to the exam but can only read 3                                 |
| Lost in the middle   | You read the first and last page but skip the middle                               |
| Stale data           | Library still has 2019 edition when 2024 is out                                    |

---

## 5. RAG vs Fine-Tuning — When to Use What?

### What is Fine-Tuning?

Fine-tuning = **Re-training the LLM on your specific data** so it "learns" your domain permanently.

### The Key Difference

```
┌───────────────────────────────────────────────────────────┐
│                                                           │
│  RAG = Give the LLM a cheat sheet at exam time            │
│        (external knowledge, at runtime)                   │
│                                                           │
│  Fine-Tuning = Make the student study extra material      │
│                beforehand (internal knowledge, at train)  │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Comparison

| Factor                | RAG                                | Fine-Tuning                         |
| --------------------- | ---------------------------------- | ----------------------------------- |
| **When data changes** | Just update the vector DB          | Must retrain the model (expensive)  |
| **Cost**              | Low (just storage + embeddings)    | High (GPU hours, training infra)    |
| **Setup time**        | Hours                              | Days to weeks                       |
| **Data freshness**    | Always up-to-date                  | Frozen at training time             |
| **Best for**          | Factual Q&A, docs, knowledge bases | Style/tone, domain behavior, format |
| **Hallucination**     | Low (grounded in retrieved docs)   | Can still hallucinate               |
| **Requires**          | Vector DB + embedding model        | Training data + GPU                 |

### When to Use What — Decision Tree

```
Do you need the LLM to know specific FACTS?
  ├── YES → Use RAG
  │         (company docs, product info, regulations)
  │
  └── NO → Does it need to BEHAVE differently?
            ├── YES → Use Fine-Tuning
            │         (write in a specific tone, follow specific format)
            │
            └── NO → Just use a good prompt
```

### Real Examples

| Use Case                               | Best Approach     | Why                             |
| -------------------------------------- | ----------------- | ------------------------------- |
| "Answer questions about our HR policy" | RAG               | Facts that change yearly        |
| "Write emails in our brand voice"      | Fine-Tuning       | Behavioral/style change         |
| "Chat about our product catalog"       | RAG               | 10,000 products, updated weekly |
| "Generate SQL for our specific schema" | RAG + Few-shot    | Schema is factual context       |
| "Act like a medical assistant"         | Fine-Tuning + RAG | Behavior + factual knowledge    |

### Can You Combine Them?

**Yes!** The best production systems often use both:

```
┌─────────────────────────────────────────────────┐
│         FINE-TUNED MODEL + RAG                   │
│                                                  │
│  Fine-tuned model knows HOW to answer            │
│  (tone, format, reasoning style)                 │
│                                                  │
│  RAG provides WHAT to answer about               │
│  (facts, docs, current data)                     │
│                                                  │
│  Result: Accurate answers in the right style     │
└─────────────────────────────────────────────────┘
```

---

## 6. RAG Architecture — Complete Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMPLETE RAG SYSTEM                           │
│                                                                 │
│  ┌─────────── OFFLINE (Indexing Pipeline) ───────────┐          │
│  │                                                   │          │
│  │  Data Sources         Processing        Storage   │          │
│  │  ┌────────┐          ┌────────┐       ┌───────┐  │          │
│  │  │  PDFs  │──┐       │ Chunk  │──────►│Vector │  │          │
│  │  ├────────┤  ├──────►│ Clean  │       │  DB   │  │          │
│  │  │  Code  │──┤       │ Embed  │       │(Pinecone│ │          │
│  │  ├────────┤  │       └────────┘       │ChromaDB)│ │          │
│  │  │  Wikis │──┘                        └───────┘  │          │
│  │  └────────┘                                      │          │
│  └───────────────────────────────────────────────────┘          │
│                                                                 │
│  ┌─────────── ONLINE (Query Pipeline) ───────────────┐          │
│  │                                                   │          │
│  │  ┌──────┐  ┌───────┐  ┌────────┐  ┌───────────┐  │          │
│  │  │ User │─►│ Embed │─►│ Search │─►│  Prompt   │  │          │
│  │  │Query │  │ Query │  │Top-K   │  │  Builder  │  │          │
│  │  └──────┘  └───────┘  └────────┘  └─────┬─────┘  │          │
│  │                                          │        │          │
│  │                                          ▼        │          │
│  │                                    ┌───────────┐  │          │
│  │                                    │    LLM    │  │          │
│  │                                    │ (Generate)│  │          │
│  │                                    └─────┬─────┘  │          │
│  │                                          │        │          │
│  │                                          ▼        │          │
│  │                                    ┌───────────┐  │          │
│  │                                    │  Answer   │  │          │
│  │                                    └───────────┘  │          │
│  └───────────────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Key Concepts — Quick Reference

### Embeddings

Embeddings convert text into numbers (vectors) so we can mathematically compare similarity.

```
"How to login?" → [0.23, -0.45, 0.89, 0.12, ...]  (1536 numbers)
"Authentication steps" → [0.21, -0.43, 0.87, 0.14, ...]  (similar vector!)
"Best pizza recipe" → [0.95, 0.32, -0.76, 0.55, ...]  (very different vector)
```

Similar meanings = similar vectors = found by search.

### Vector Database

A database optimized for finding "nearest neighbors" — i.e., which stored vectors are most similar to your query vector.

**Popular options:** Pinecone, ChromaDB, Weaviate, Qdrant, pgvector (PostgreSQL extension)

### Chunking Strategies

| Strategy       | How it Works                              | Best For                     |
| -------------- | ----------------------------------------- | ---------------------------- |
| Fixed-size     | Split every N tokens                      | Simple docs                  |
| Sentence-based | Split at sentence boundaries              | Articles, blogs              |
| Semantic       | Split by topic/meaning change             | Technical docs               |
| Recursive      | Try large splits, then smaller if too big | Code, mixed content          |
| Document-based | One chunk per document                    | Short docs (emails, tickets) |

### Temperature

Controls how "creative" vs "factual" the LLM is:

- **Low (0.0-0.3):** Deterministic, factual — use for RAG answers
- **High (0.7-1.0):** Creative, varied — use for brainstorming

For RAG, **always use low temperature** (you want factual, grounded answers).

---

## 8. Building a Simple RAG — Mental Model

If you were to code a basic RAG system, here's the pseudocode:

```javascript
// PHASE 1: Indexing (run once)
const documents = loadDocuments("./my-data/");
const chunks = splitIntoChunks(documents, { size: 500, overlap: 50 });
const vectors = await embedAll(chunks); // using OpenAI/Cohere embeddings
await vectorDB.store(vectors);

// PHASE 2: Querying (run every question)
async function askRAG(question) {
  // Step 1: Embed the question
  const queryVector = await embed(question);

  // Step 2: Find similar chunks
  const relevantChunks = await vectorDB.search(queryVector, { topK: 5 });

  // Step 3: Build the prompt
  const prompt = `
    Based on the following context, answer the question.
    If the answer is not in the context, say "I don't know."

    CONTEXT:
    ${relevantChunks.map((c) => c.text).join("\n\n")}

    QUESTION: ${question}
  `;

  // Step 4: Generate answer
  const answer = await llm.generate(prompt);
  return answer;
}
```

---

## Summary Table

| Concept       | One-Line Explanation                          |
| ------------- | --------------------------------------------- |
| RAG           | Search your docs first, then ask the LLM      |
| Embedding     | Convert text to numbers for similarity search |
| Vector DB     | Database that finds "similar" vectors fast    |
| Chunking      | Splitting docs into bite-sized pieces         |
| Re-ranking    | Scoring retrieved chunks by actual relevance  |
| Fine-tuning   | Permanently teaching LLM a new behavior/style |
| Naive RAG     | Simple retrieve → generate pipeline           |
| Advanced RAG  | Adds query rewriting + re-ranking + filtering |
| Agentic RAG   | LLM decides what to search and when           |
| Hallucination | LLM confidently making stuff up               |

---

## Homework

1. Pick any PDF (your college syllabus, a textbook chapter)
2. Manually split it into 5-10 chunks
3. For a given question, pick which chunks are most relevant (you are the "vector search")
4. Write a prompt: "Based on this context: [chunks], answer: [question]"
5. Paste it into ChatGPT/Claude and see how the answer quality improves vs asking without context

This is RAG — you just did it manually. Tools like LangChain and LlamaIndex automate these steps.
