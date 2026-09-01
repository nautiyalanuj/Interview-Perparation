# Using Vectors in PostgreSQL (pgvector)

One of the most common use cases for vectors in PostgreSQL is **semantic search** for a RAG (Retrieval-Augmented Generation) system.

---

# Step 1: Original Data

Imagine you have a table of support articles:

| Id | Content |
|------|---------|
| 1 | Reset your password from the settings page |
| 2 | Update billing information in account settings |
| 3 | Change your email address from profile settings |

A traditional SQL search might look like:

```sql
SELECT *
FROM articles
WHERE content LIKE '%password%';
```

This only works if the word **password** appears in the document.

---

# Step 2: Generate Embeddings

An embedding model converts each piece of text into a numeric vector.

Example:

```text
"Reset your password from the settings page"
→ [0.21, 0.43, -0.55, ...]

"Update billing information"
→ [-0.88, 0.72, 0.11, ...]

"Change email address"
→ [0.09, -0.34, 0.66, ...]
```

Modern models typically generate:

```text
1536 dimensions
```

or

```text
3072 dimensions
```

Each dimension represents a learned semantic feature.

---

# Step 3: Store in PostgreSQL

Install and enable pgvector:

```sql
CREATE EXTENSION vector;
```

Create table:

```sql
CREATE TABLE articles (
    id BIGSERIAL PRIMARY KEY,
    content TEXT,
    embedding VECTOR(1536)
);
```

Insert data:

```sql
INSERT INTO articles(content, embedding)
VALUES (
  'Reset your password from the settings page',
  '[0.21,0.43,-0.55,...]'
);
```

Stored data:

```text
+----+--------------------------------------+-------------------+
| id | content                              | embedding         |
+----+--------------------------------------+-------------------+
| 1  | Reset password article               | [0.21,0.43,...]   |
| 2  | Billing article                      | [-0.88,0.72,...]  |
| 3  | Email update article                 | [0.09,-0.34,...]  |
+----+--------------------------------------+-------------------+
```

The vector is stored alongside the document.

---

# Step 4: User Asks a Question

Suppose the user asks:

```text
"I forgot my login credentials"
```

Generate an embedding for the query:

```text
"I forgot my login credentials"
→ [0.24,0.39,-0.49,...]
```

Notice something important:

```text
User Query:
"forgot login credentials"

Document:
"reset password"
```

The words are different, but semantically they mean something very similar.

Traditional SQL keyword search may miss this relationship.

---

# Step 5: Similarity Search

PostgreSQL compares the query vector to all stored vectors.

Example similarity scores:

| Article | Similarity |
|-----------|-----------|
| Reset password article | 0.95 |
| Billing article | 0.18 |
| Email update article | 0.31 |

Query:

```sql
SELECT id, content
FROM articles
ORDER BY embedding <=> :query_embedding
LIMIT 3;
```

Result:

```text
Reset your password from the settings page
```

Even though the word **password** never appeared in the user's query.

This is called **semantic search**.

---

# What Happens Mathematically?

Assume tiny 3-dimensional vectors:

```text
Password Article:

[1, 2, 3]

User Query:

[1.1, 2.1, 3.0]
```

These vectors point in almost the same direction.

A distance function such as:

- Cosine Distance
- Euclidean Distance
- Dot Product

computes how close they are.

Another article:

```text
[100, 50, 20]
```

would have a much larger distance.

PostgreSQL ranks results by nearest distance.

---

# Indexing for Performance

Without an index:

```text
10 million vectors
```

requires scanning every row.

To speed this up:

```sql
CREATE INDEX article_embedding_idx
ON articles
USING hnsw (embedding vector_cosine_ops);
```

This creates an **HNSW (Hierarchical Navigable Small World)** index.

Instead of:

```text
Scan every vector
```

PostgreSQL does:

```text
Traverse HNSW graph
↓
Find nearest neighbors
↓
Return top K matches
```

This dramatically improves query performance.

---

# Real RAG Example

Suppose we have:

```sql
CREATE TABLE documents (
    id BIGSERIAL,
    tenant_id INT,
    title TEXT,
    content TEXT,
    embedding VECTOR(1536)
);
```

Stored Documents:

| Title |
|---------|
| Terraform State Migration Guide |
| Terraform Import Instructions |
| Terraform Backend Setup |

---

## User Question

```text
How do I import terraform state?
```

---

## Step 1: Generate Query Embedding

```csharp
var embedding = await openAI.GetEmbeddingAsync(
    "How do I import terraform state?");
```

---

## Step 2: Search PostgreSQL

```sql
SELECT title,
       content
FROM documents
WHERE tenant_id = 100
ORDER BY embedding <=> :queryEmbedding
LIMIT 5;
```

---

## Step 3: Retrieve Relevant Chunks

Results:

```text
Terraform State Migration Guide
Terraform Import Instructions
Terraform Backend Setup
```

---

## Step 4: Send Context to LLM

Prompt:

```text
Question:
How do I import terraform state?

Context:
<retrieved documents>

Answer using only the provided context.
```

This is the classic **RAG pipeline**.

---

# Why PostgreSQL + pgvector Is Popular

Without pgvector:

```text
PostgreSQL
+
Separate Vector Database
+
Data Synchronization Layer
```

With pgvector:

```text
PostgreSQL
├─ Users
├─ Orders
├─ Documents
├─ Metadata
└─ Embeddings
```

Everything resides in a single database.

Benefits:

- Simpler architecture
- Lower operational overhead
- SQL + vector search in one place
- Transactional consistency
- Easier filtering with metadata

---

# Mental Model

```text
Traditional Search

User Query
    ↓
Keyword Match
    ↓
Results
```

```text
Vector Search

User Query
    ↓
Embedding Model
    ↓
Query Vector
    ↓
Similarity Search
    ↓
Top Matching Documents
```

The key idea is:

> PostgreSQL stores both the original document and its embedding. When a user asks a question, the question is converted into an embedding, and PostgreSQL finds the stored vectors that are semantically closest to the query vector.
