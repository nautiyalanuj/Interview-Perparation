## What is celebrity problem?
- Too many request for a hot key
- And what happen if a hot key expires?

## How do we identify hot keys?
- We track request frequency using metrics or streaming analytics. Keys exceeding a request threshold over a sliding time window are classified as hot.
- We can some hot key detector service.

## How does each server knows get those hot keys?
  - Server can get data by using pub/sub for redis, so hot key detector keep on pushing the latest data.
  - Or calling hot key detector periodically in background
  - **Never put anything on the critical request path that can be cached locally.**
    - Hot Key Metadata, routing tables, shard maps, feature flags, and hot-key information are usually pushed to servers rather than fetched on every request.

## Now each server knows hot keys, how does duplicating keys work now??
- So now server know which are the hot keys,so whenever request come in case of duplication, a server can randomly ask any shard for the data. 

## In case of expiry how cache stampede problem is solved?
- Request coalescing protects the database when a hot key experiences a cache miss.
- Here also we need to identify hot keys as mentioned above and send only few request rather than bunch of request.

## Why not proactively do refresh??
- Imagine 500 million user and we proactively refresh every key. We would destroy your database as most of the key will never be read again.
- Thus, background refresh works best for hot keys.

## But as you mentioned above we can do proactive refresh, then do we need  request coalescing?
- What happen if say redis restarted?
- What if background refresh somehow fails?
- what is the key somehow got evicted/deleted?
