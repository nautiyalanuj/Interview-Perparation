# Interview
When discussing a rate limiter, mention these four concerns:
1. Correctness
   - Thread safety
   - Atomic updates

2. Scalability
   - Distributed deployment
   - Redis + Lua scripts

3. Memory Growth
   - Millions of users create millions of buckets
   - Use TTL / eviction

4. Traffic Pattern
   - Token Bucket supports bursts
   - Long-term rate still enforced
  
# Token Bucket (Most popular)
  - Allow burst as if you haven't used your quota recently, you may spend saved tokens in a burst.
  - Simple to implement, basically fill the bucket periodically.
    - One of the way is to keep on filling every time a request is received. So, in this way you can have entry as double also i.e. 4.5 request is allowed. This avoid keeping some external background service to maintain the bucket.
  - Every request:
    - Read current bucket
    - Calculate refill
    - Update tokens
    - Consume token
    - Save bucket
 - Memory Growth Problem
   
   Suppose we're maintaining:
   
   ```ConcurrentHashMap<String, TokenBucket> buckets;```

    where key = userId.Thus, every time a new user arrives we creates a new bucket. After a month, buckets.size() == 10_000_000, even if most users never come back, their buckets remain in memory.

   - Solution
     - TTL
     - Scheduled Cleanup
# Sliding Window
  - Pros
    - Less memory
    - Good Accuracy
  - Cons
    - More complex

# Fixed Window (Not Preferred)
  - Cons
    - Boundary problem, as fix window what if request comes at 10:59 and 11:01, so per minute boundary for every interval is not supported.
  - Cons
    - Simple and Fast

## Distributed system solution
- For production same is preferred choice, here we can choose any technology which we want mostly redis is used for same.
  - For redis
    - Redis is fast, in-memory and provide atomic operation thus suits best.
    - Support lua script to handle race condition like below in case of token bucket.
```
App-1 reads 5 tokens
App-2 reads 5 tokens

Both consume 1

Expected: 3 tokens
Actual: 4 tokens
```
    
## Single Machine Rate Limiter
- Use simple lock for managing critical section within one user/API request. So, if a 100 request per user/API use locks;
- For supporting multiple user/API use concurrenthashmap. So, supporting 100 request for multiple user/API.

# Client Side Rate Limiting
- Client-side rate limiting improves user experience and reduces unnecessary traffic, but it should never be the primary protection mechanism because it is easy to bypass, cannot enforce global quotas, cannot prevent malicious traffic, and has no authoritative view across users, tenants, or devices. Therefore, production systems typically implement server-side rate limiting and use client-side rate limiting only as an optimization layer.
- Benefits
  - Reduces unnecessary traffic
  - Prevents accidental request floods
  - Gives faster user feedback
  - Reduces 429 responses
  - Lowers server load
- Example:
  - A search box sends requests only once every 300 ms instead of on every keystroke.

