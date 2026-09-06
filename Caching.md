# When to to use caching
- Reduce latency
- Database is the bottleneck => DB becomes overloaded
- Data is expensive to compute
  - Request -> Complex query -> 5 microservice calls

# Alternative to caching
- Database indexing.
  - Move from slow query + Redis => Optimized query + Index
- Read replicas => Good when the problem is database read scale
- Materialized views / precomputed results
  - Instead of recalculating expensive queries: store results periodically.
  - Great for Dashboards, Reporting and Analytics
- Data De-normalization
- Instead of 10 joins, store data in a shape optimized for reading

# When not to use cache
 - Data changes very frequently
   - Stock trading price
 - Strong consistency is required
   - Bank account balances, Payment processing and Inventory during checkout
- Large data sets
  - If objects are huge and memory usage becomes expensive, caching may not be economical.
  - Usually CDNs or object storage are better.
- Low cache hit ratio
  - If users rarely request the same data, most requests will be cache misses

# Problem with cache 
- Cache consistency problems
- Cache stampede / thundering herd => Cache expires, Database suddenly receives huge load.
  - If this is for one key, i.e. a key expire and 10k request comes for that key and try to call db and update same 
    - Solve same using distributed locking
    - Serve stale and do background refresh
    - Other way to refresh cache is based on expiry time, so create a probability function which check if a key is near expiry it will be fetched again. So, only one of the call will get affected. So, if I a request comes and 20 minute is remaining, lets have 1 %chance to refresh, for 15 min 20% chance and so nm.
  - 1 million key with same near-time expiry
    - Randomized TTL
    - Refresh ahead
  - Cold start problem => Cache starts empty
    - Cache Pre-warming is one of the solution

# Type of caching
- Client caching
  - Caches can be located on the client side (OS or browser), server side, or in a distinct cache layer.
- CDN caching
  - CDNs are considered a type of cache.
- Web server caching
  - Reverse proxies and caches such as Varnish can serve static and dynamic content directly. Web servers can also cache requests, returning responses without having to contact application servers. 
- Database caching
  - Your database usually includes some level of caching in a default configuration, optimized for a generic use case. Tweaking these settings for specific usage patterns can further boost performance.
  - Cons
    - Databases generally cache raw data blocks or table rows. If your application needs to perform heavy logic on that data—such as parsing complex JSON, running calculations, joining multiple tables, or constructing a deep object tree—the application has to re-run that expensive CPU work every single time, even if the raw rows are instantly pulled from the database RAM. An application cache stores the final, fully computed result.
    - Databases are structurally limited by the number of concurrent connections they can handle. Every active query uses a database thread. If 10,000 users request the same data at the exact same second, hitting the database directly—even just its memory—can exhaust the connection pool and crash your application.
- Application caching 
  - In-memory caches such as Memcached and Redis are key-value stores between your application and your data storage. Since the data is held in RAM, it is much faster than typical databases where data is stored on disk. RAM is more limited than disk, so cache invalidation algorithms such as least recently used (LRU) can help invalidate 'cold' entries and keep 'hot' data in RAM. 
