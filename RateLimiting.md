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

# Server Side Rate Limiting
