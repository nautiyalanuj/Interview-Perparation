# Java HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap - Notes

---

# Map Implementations Overview

| Feature | HashMap | LinkedHashMap | TreeMap | ConcurrentHashMap |
|----------|----------|-------------|---------|------------------|
| Ordering | No guarantee | Insertion order / Access order | Sorted by key | No guarantee |
| Lookup Complexity | O(1) avg | O(1) avg | O(log n) | O(1) avg |
| Thread Safe | No | No | No | Yes |
| Null Key | 1 allowed | 1 allowed | Not allowed | Not allowed |
| Null Value | Allowed | Allowed | Allowed | Not allowed |
| Internals | Hash Table | Hash Table + Doubly Linked List | Red-Black Tree | Concurrent Hash Table |
| Typical Usage | Fast lookup | Ordered iteration, LRU cache | Sorted data | Concurrent access |

---
# Functions
- Map<String, Integer> m = new HashMap<>(); 
- m.put("a", 1); 
- m.get("a");                 // 1 
- m.getOrDefault("z", 0);     // 0 
- m.containsKey("a"); 
- m.remove("a"); 
- m.size(); 
- m.putIfAbsent("a", 1); 
- Traversal 
```
for (Map.Entry<String, Integer> entry : map.entrySet())
{
        System.out.println("Key: " + entry.getKey() + ", Value: " + entry.getValue());
} 
```
---

# Why Resize?

Suppose:

```java
capacity = 4
```

and many keys map to the same bucket.

```text
bucket[1]

1 -> 5 -> 9 -> 13
```

Lookup:

```java
get(13)
```

Requires traversal:

```text
1 -> 5 -> 9 -> 13
```

As bucket chains grow, performance degrades.

---

## Goal of Resize

Spread entries across more buckets.

Example:

Before:

```text
capacity = 4
```

After:

```text
capacity = 8
```

Now:

```text
bucket[1] -> 1 -> 9
bucket[5] -> 5 -> 13
```

Shorter chains.

Faster lookups.

---

# What Happens During Resize?

Suppose:

```java
capacity = 16
threshold = 12
```

Insert 13th element.

Java performs:

### Step 1

Allocate larger bucket array

```text
16 -> 32 buckets
```

### Step 2

Visit every old bucket

```text
bucket 0
bucket 1
...
bucket 15
```

### Step 3

Recalculate bucket index

```java
hash & (newCapacity - 1)
```

### Step 4

Move entries into new buckets

### Step 5

Replace table reference

```java
table = newTable;
```

---

# Why Not Use Consistent Hashing?

Consistent Hashing solves a different problem.

---

## HashMap Problem

Distribute keys over memory buckets:

```java
bucket = hash(key) % capacity
```

Example:

```text
bucket 0
bucket 1
bucket 2
...
```

---

## Consistent Hashing Problem

Distribute keys across machines.

Example:

```text
User1 -> Server A
User2 -> Server B
User3 -> Server C
```

When a new server joins:

```text
Server D
```

Only a subset of keys move.

Used in:

- Cassandra
- DynamoDB
- Redis Cluster
- Distributed caches

Not useful inside a single JVM hashmap.

---

# Resize Cost

Normal operation:

```java
put()
```

Average:

```text
O(1)
```

Resize operation:

```text
O(n)
```

because every entry must be relocated.

---

## Latency Impact

Example:

```text
999,999 inserts -> fast
1 insert -> causes resize -> expensive
```

The resizing insert may take significantly longer.

This is why we say:

```text
HashMap operations are amortized O(1)
```

not strictly O(1).

---

# What Happens To Requests During Resize?

## HashMap

Single-threaded.

Thread performing:

```java
put()
```

must finish resize before continuing.

Other operations wait.

---

## ConcurrentHashMap

Java 8+ uses cooperative resizing.

Example:

```text
Thread A starts resize
Thread B helps
Thread C helps
```

Multiple threads participate in migration.

Resizing impact is reduced.

Operations can continue concurrently.

---

# Why Doesn't HashMap Automatically Shrink?

Consider:

```java
insert 1,000,000 entries
```

Then:

```java
remove 999,000 entries
```

Should capacity shrink?

Java says:

```text
No
```

Reason:

Frequent growing and shrinking causes:

```text
capacity thrashing
```

Example:

```text
grow
shrink
grow
shrink
```

which can be very expensive.

Therefore:

- HashMap grows automatically
- HashMap generally does not shrink automatically

Same for:

- LinkedHashMap
- ConcurrentHashMap

---

# LRU Cache with LinkedHashMap

Use access order:

```java
new LinkedHashMap<>(16, 0.75f, true);
```

Override:

```java
@Override
protected boolean removeEldestEntry(
        Map.Entry<K,V> eldest) {
    return size() > capacity;
}
```

Example:

```java
put(1)
put(2)
put(3)

get(1)

put(4)
```

Order before inserting 4:

```text
2 -> 3 -> 1
```

Evict:

```text
2
```

Final cache:

```text
3 -> 1 -> 4
```

---

# Interview Takeaways

### HashMap

Use when:

- Fast lookup needed
- No ordering required

---

### LinkedHashMap

Use when:

- Iteration order matters
- Building LRU cache

---

### TreeMap

Use when:

- Keys must remain sorted
- Range queries needed

```java
floorKey()
ceilingKey()
headMap()
tailMap()
```

---

### ConcurrentHashMap

Use when:

- Multiple threads access the map
- High throughput concurrent reads/writes required

---

# Senior Backend Takeaway

Hash table resizing is a tradeoff:

Without resizing:

```text
More collisions
Slower lookups
```

With resizing:

```text
Occasional O(n) latency spikes
Better long-term performance
```

For large production systems, pre-size maps when expected cardinality is known:

```java
new ConcurrentHashMap<>(10_000_000);
```

to avoid expensive resize operations during peak traffic

# LinkedHashMap Constructor

```java
new LinkedHashMap<>(16, 0.75f, true);
```

Equivalent to:

```java
LinkedHashMap(
    int initialCapacity,
    float loadFactor,
    boolean accessOrder
)
```

---

## Parameter 1: Initial Capacity

```java
16
```

Initial bucket count.

```text
bucket[0]
bucket[1]
...
bucket[15]
```

Threshold:

```java
capacity * loadFactor
```

```java
16 * 0.75 = 12
```

The 13th insertion triggers resize.

---

## Parameter 2: Load Factor

```java
0.75f
```

Determines how full the hash table can become before resizing.

Formula:

```java
threshold = capacity * loadFactor
```

Example:

```java
capacity = 16
loadFactor = 0.75

threshold = 12
```

When size > 12:

```java
resize()
```

### Tradeoffs

Lower Load Factor

```java
0.5
```

Pros:
- Fewer collisions
- Faster lookups

Cons:
- More memory usage

Higher Load Factor

```java
1.0
```

Pros:
- Less memory

Cons:
- More collisions

Default:

```java
0.75
```

Good balance between memory and performance.

---

## Parameter 3: Access Order

```java
true
```

Controls ordering of iteration.

---

### Insertion Order

```java
new LinkedHashMap<>(16, 0.75f, false);
```

```java
put(1)
put(2)
put(3)

get(1)
```

Order remains:

```text
1 -> 2 -> 3
```

---

### Access Order

```java
new LinkedHashMap<>(16, 0.75f, true);
```

```java
put(1)
put(2)
put(3)

get(1)
```

Order becomes:

```text
2 -> 3 -> 1
```

Most recently accessed entry moves to the tail.

---

# LinkedHashMap Internals

LinkedHashMap is NOT just a linked list.

It contains:

```text
Hash Table
      +
Doubly Linked List
```

---

## Hash Table Part

Used for:

```java
get(key)
put(key)
remove(key)
```

Structure:

```text
bucket[0]
bucket[1]
bucket[2]
...
```

Provides:

```text
O(1) average lookup
```

Load factor and resizing affect this part.

---

## Doubly Linked List Part

Used for:

- Insertion ordering
- Access ordering
- LRU caches

Structure:

```text
head
 ↓
1 <-> 2 <-> 3 <-> 4
                 ↑
               tail
```

Load factor has no effect here.

---
