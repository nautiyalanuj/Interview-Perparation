# Supported
- Stack
  - ArrayDeque  (resizable, circular array)(default)
  - Stack (Legacy)
- Queue
  - ArrayDeque   (resizable, circular array)(default)
- Double link List /Linked List
  - ArrayDeque  (resizable, circular array)(default)
  - LinkedList
- Map as HashMap 
- Set as HashSet
- Heap as PriorityQueue
- Array
  - 2d array
- Balanced Tree
  - TreeSet
  - TreeMap

# Concurrent 
- ConcurrentHashMap
  - The go-to concurrent map. It uses bucket-level locking (and CAS operations) to allow multiple threads to read and write simultaneously without blocking each other. 
- ConcurrentSkipListMap
  - A thread-safe, scalable, sorted map implementation. It orders keys naturally or via a custom Comparator, offering \(O(\log n)\) time cost for operations.
- ConcurrentLinkedQueue
  - An unbounded, thread-safe FIFO queue based on linked nodes using non-blocking CAS algorithms.
- ConcurrentLinkedDeque
  - An unbounded, non-blocking double-ended queue using CAS algorithms.
- ConcurrentSkipListSet:
  - A thread-safe, sorted set. Under the hood, it is backed by a ConcurrentSkipListMap. 
  
# Not Supported
- Trie

