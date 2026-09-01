# Problem => Race Conditions

If two threads execute these steps simultaneously, updates can be lost. This is called a **race condition**.


The operation:

```java
count++;
```

is internally:

```text
1. Read count
2. Increment value
3. Write updated value
```

---

# Interview-Friendly Answer

> Java handles concurrency using threads, synchronization mechanisms (`synchronized`, `Lock`), memory visibility controls (`volatile`), atomic operations (`AtomicInteger`), and high-level concurrency utilities from `java.util.concurrent` such as `ExecutorService`, `CompletableFuture`, `ConcurrentHashMap`, and `BlockingQueue`.

---

# 1. Synchronization (`synchronized`)

Java provides intrinsic locks (monitors) through the `synchronized` keyword.

```java
class Counter {
    private int count = 0;

    synchronized void increment() {
        count++;
    }
}
```

Equivalent:

```java
void increment() {
    synchronized (this) {
        count++;
    }
}
```

### Benefits

- Mutual exclusion.
- Prevents race conditions.
- Ensures memory visibility.

### Drawback

- Only one thread can enter the synchronized block at a time.

---

# 2. Memory Visibility and `volatile`

Even if only one thread writes to a variable, other threads may not immediately see the updated value because of CPU caching.

```java
boolean running = true;

while (running) {
    // may never stop
}
```

Solution:

```java
volatile boolean running = true;
```

### What `volatile` Guarantees

- Latest value is always visible to all threads.
- Reads and writes are performed directly from/to main memory.

### What `volatile` Does NOT Guarantee

It does not make compound actions atomic.

```java
volatile int count;

count++; // Still NOT thread-safe
```

---

# 3. Atomic Variables

For lock-free thread-safe operations, Java provides atomic classes.

```java
AtomicInteger counter = new AtomicInteger();

counter.incrementAndGet();
```

### Common Atomic Classes

```java
AtomicInteger
AtomicLong
AtomicBoolean
AtomicReference
```

### Advantages

- Faster than locks for simple updates.
- Uses Compare-And-Swap (CAS) at the CPU level.

---

# 4. Explicit Locks

Java provides more flexible locking APIs via `Lock`.

```java
Lock lock = new ReentrantLock();

lock.lock();

try {
    // Critical section
} finally {
    lock.unlock();
}
```

### Advantages over `synchronized`

```java
lock.tryLock();
lock.lockInterruptibly();
```

Supports:

- Timed lock acquisition.
- Interruptible locking.
- Fair locks.

---

# 5. ReadWriteLock

Useful when reads are much more frequent than writes.

```java
ReadWriteLock rwLock =
    new ReentrantReadWriteLock();
```

### Read Operation

```java
rwLock.readLock().lock();

try {
    // Read data
} finally {
    rwLock.readLock().unlock();
}
```

### Write Operation

```java
rwLock.writeLock().lock();

try {
    // Modify data
} finally {
    rwLock.writeLock().unlock();
}
```

### Behavior

✅ Multiple readers allowed.

❌ Only one writer allowed.

---
# Utilities

## Concurrent Collections

Standard collections are not thread-safe.

Use concurrent alternatives:

```java
ConcurrentHashMap
CopyOnWriteArrayList
ConcurrentLinkedQueue
BlockingQueue
```

### Example

```java
Map<Integer, String> map =
    new ConcurrentHashMap<>();

map.put(1, "Java");
```

### Benefits

- Safe concurrent access.
- Higher throughput than synchronized collections.


## BlockingQueue (Producer-Consumer)

A common concurrency pattern.

```java
BlockingQueue<Task> queue =
    new LinkedBlockingQueue<>();
```

### Producer

```java
queue.put(task);
```

### Consumer

```java
Task task = queue.take();
```

### Uses

- Thread pools.
- Job processing systems.
- Message queues.

---

# Quick Summary

| Problem | Java Solution |
|----------|---------------|
| Protect shared state | synchronized, Lock |
| Memory visibility | volatile |
| Atomic updates | AtomicInteger |
| Read-heavy workloads | ReadWriteLock |
| Thread-safe collections | ConcurrentHashMap |
| Producer-consumer workflow | BlockingQueue |

---

