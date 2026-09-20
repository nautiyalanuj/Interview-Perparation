# PostGres as swiss knife

- Custom data types, functions, operators and extensions
- Support text search using GIN and tsvector
- Support json using GIN and jsonb
- Support Geospatial Search with PostGIS
- Support vector search for AI.
- Support Timeseries extension which is TimescaleDB  => PostgreSQL + Time-series extensions
  -  Alternative for prometheus
  -  Full SQL support
  -  Joins
  -  Relational data + metrics together
  -  Works excellently with Grafana.
 
# Text Search
### Native Postgres (`tsvector` + GIN) vs. `pg_search` (BM25 via ParadeDB)

| Feature | **Native Postgres (`tsvector` + GIN)** | **`pg_search` (ParadeDB)** |
| --- | --- | --- |
| **Availability** | **Built-in** to all PostgreSQL databases. Zero extensions or setup required. | Requires installing the **`pg_search` extension** (often restricted on managed hosts like standard AWS RDS). |
| **Scoring Algorithm** | `ts_rank` (simple frequency / location-based density scoring). | **BM25** (industry standard used by Lucene / Elasticsearch). |
| **Top-N / Ordering Speed** | **Slower at scale.** GIN fetches matches, but Postgres must score *every match* on the heap before sorting. | **Significantly faster.** The BM25 index stores term statistics natively, allowing fast Top-$N$ retrievals. |
| **Search Features** | Exact lexeme matching; basic stemming and prefix matching. | Advanced features out-of-the-box: fuzzy search, typo tolerance, highlighting, and faceting. |
| **Storage & Writes** | Smaller index footprint. Extremely fast exact counts. | Slightly larger index size; fast creation via Rust/Tantivy engine. |

---

### When to Choose Native `tsvector` + GIN

Choose standard Postgres full-text search if:

1. **Zero External Dependencies / Restricted Host:** You run on an environment where custom C/Rust extensions are forbidden or unavailable (e.g., standard managed cloud instances without custom extension support).
2. **Boolean / Filtering Search over Ranking:** Your application cares mostly about *matching* criteria (e.g., "find all articles where `category = 'news'` and text contains 'postgres'") rather than finding the top-10 fuzzy relevance matches.
3. **Exact Counts Matter:** You frequently run queries like `COUNT(*)` over large text search matches—native GIN scans are drastically faster for exact matching counts.
4. **Small-to-Medium Datasets:** Your target tables have under 1–2 million rows. At this scale, `ts_rank` and GIN performance are usually well within acceptable web SLAs.

---

### When to Choose `pg_search`

Choose `pg_search` if:

1. **Relevance Ranking is Critical:** You are building an user-facing search bar (e.g., e-commerce, documentation search, or SaaS search) where users expect "Google-like" or "Elasticsearch-like" relevant results at the top.
2. **High Scale & Large Datasets:** You are querying millions of rows where sorting by `ORDER BY ts_rank DESC LIMIT 10` causes native GIN queries to time out.
3. **You Want to Replace Elasticsearch:** You want modern search features (typo tolerance, fuzzy matching, highlighting) without needing an ETL pipeline or maintaining a separate search cluster.
4. **Hybrid Search (with `pgvector`):** You are building AI/RAG applications that combine BM25 keyword search with vector embeddings.
