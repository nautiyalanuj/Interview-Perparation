# v4
- Generation Mechanism: Fully random bit layout (122 bits of pure cryptographic entropy).
- Primary Use Case: Default general-purpose identifier (e.g., API keys, session tokens, transaction tracking).
- Status: Highly Popular. Excellent for decoupling data context entirely.

# v5
- Generation Mechanism: Deterministic SHA-1 hash of a chosen Namespace + Name string.
- Primary Use Case: Cross-system data integration, creating consistent IDs for natural keys (URLs, emails).
- Status: Active Standard. The preferred choice for deterministic, repeatable hashing.

# v7
- Generation Mechanism: 48-bit Unix Epoch Millisecond timestamp + ~74 bits of random entropy.
- Primary Use Case: Modern database primary keys (B-Tree friendly, eliminates index fragmentation).
- Status: Recommended Default for relational and NoSQL databases.
