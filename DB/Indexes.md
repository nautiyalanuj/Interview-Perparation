# Type of Indexes
This guide is prepared with help of hellointerview

## No Index
  - Don't index if there is no need.
    
## B+ Tree Hash
  - General purpose index in all SQL Db
  - Traditional indexes like B-trees don't work well for spatial data because they treat latitude and longitude as independent dimensions. To efficiently search for nearby locations, we need an index that understands spatial relationships

## Geospatial Hash
  - For location proximity searches like used in Uber
  - There are three main approaches you'll encounter in interviews: geohashes, quadtrees, and R-trees. Each has its own strengths and trade-offs, but all solve our fundamental problem: they preserve spatial relationships in our index structure.
  - Geohash is a hash-based approach that converts 2D coordinates into a 1D string, preserving proximity. This allows us to use a regular B-tree index on the geohash strings for efficient proximity searches.  However, tree-based approaches like R-trees can offer more flexibility and accuracy by grouping nearby objects into overlapping rectangles, creating a hierarchy of bounding boxes
  - Geospatial Search with PostGIS. While not built into PostgreSQL core, the PostGIS extension adds powerful spatial capabilities.

## Inverted Index
  - While B-trees excel at finding exact matches and ranges, they fall short when we need to search through text content. Consider what happens when you run a query like:
  - ``` SELECT * FROM posts WHERE content LIKE '%database%';```
    - Here, we're looking for posts that contain the word "database" anywhere in their content - not just posts that start or end with it. Even with a B-tree index on the content column, the database can't use the index at all. Why? B-tree indexes can only help with prefix matches (like 'database%') or suffix matches (if you index the reversed content). When the pattern could match anywhere within the text, the database has no choice but to check every character of every post, reading entire text fields into memory to look for matches. 
  - Text search like used in elastic-search
  - Postgres supports full-text search out of the box using GIN (Generalized Inverted Index) indexes. GIN indexes work like the index at the back of a book - they store a mapping of each word to all the locations where it appears. 

## Hash Tree Index
  - In-memory index, basically key-value pair used in redis/memcache.
