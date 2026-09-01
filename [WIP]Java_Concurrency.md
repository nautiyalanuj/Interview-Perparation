# How Concurrency is Handled in Java

Concurrency in Java is handled through a combination of:

1. **Threads** for parallel execution.
2. **Synchronization mechanisms** for protecting shared resources.
3. **Memory visibility controls** to ensure changes are visible across threads.
4. **High-level concurrency utilities** provided by `java.util.concurrent`.

At a high level, Java allows multiple tasks to execute concurrently while providing mechanisms to prevent race conditions and maintain data consistency.

---

# 1. Threads: The Foundation

A thread is the smallest unit of execution in Java.

## Creating a Thread

```java
class MyTask extends Thread {
    @Override
    public void run() {
        System.out.println("Task running");
    }
}

public class Main {
    public static void main(String[] args) {
        MyTask thread = new MyTask();
        thread.start();
    }
}
```

## Using Runnable (Preferred)

```java
Runnable task = () -> System.out.println("Task running");

Thread thread = new Thread(task);
thread.start();
```

---

# 2. Race Conditions

When multiple threads access and modify shared data simultaneously, race conditions can occur.

```java
class Counter {
    int count = 0;

    void increment() {
        count++;
    }
}
```

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

If two threads execute these steps simultaneously, updates can be lost.

### Example

Expected:

```text
2000
```

Possible actual result:

```text
1873
```

This is called a **race condition**.

---

# 3. Synchronization (`synchronized`)

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

# 4. Memory Visibility and `volatile`

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

# 5. Atomic Variables

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

# 6. Explicit Locks

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

# 7. Thread Pools

Creating threads repeatedly is expensive.

Instead, use a thread pool:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(10);

executor.submit(() -> {
    System.out.println("Task running");
});
```

### Benefits

- Reuses threads.
- Improves performance.
- Controls resource usage.

---

# 8. Callable and Future

`Runnable` cannot return a value.

Use `Callable` for tasks that return results.

```java
Callable<Integer> task = () -> 42;

Future<Integer> future =
    executor.submit(task);

Integer result = future.get();
```

### Future Features

```java
future.isDone();
future.cancel(true);
future.get();
```

---

# 9. CompletableFuture

Modern asynchronous programming in Java.

```java
CompletableFuture
    .supplyAsync(() -> fetchData())
    .thenApply(data -> process(data))
    .thenAccept(System.out::println);
```

### Parallel Example

```java
CompletableFuture<User> userFuture =
        CompletableFuture.supplyAsync(
            () -> loadUser());

CompletableFuture<List<Order>> orderFuture =
        CompletableFuture.supplyAsync(
            () -> loadOrders());

CompletableFuture.allOf(
    userFuture,
    orderFuture
).join();
```

### Advantages

- Non-blocking workflows.
- Easy async composition.
- Better readability than nested callbacks.

---

# 10. Concurrent Collections

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

---

# 11. BlockingQueue (Producer-Consumer)

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

# 12. ReadWriteLock

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

# 13. Fork/Join Framework

Designed for divide-and-conquer algorithms.

```java
ForkJoinPool pool =
    new ForkJoinPool();
```

### Workflow

```text
Split Task
    ↓
Execute Subtasks
    ↓
Combine Results
```

Commonly used for:

- Parallel computations.
- Recursive algorithms.
- Data processing.

Used internally by:

```java
parallelStream()
```

---

# 14. Virtual Threads (Java 21+)

Virtual threads are lightweight threads managed by the JVM.

```java
Thread.startVirtualThread(() -> {
    doWork();
});
```

### Executor Example

```java
try (var executor =
         Executors.newVirtualThreadPerTaskExecutor()) {

    executor.submit(() -> callService());
}
```

### Benefits

- Millions of concurrent tasks.
- Low memory footprint.
- Ideal for I/O-heavy applications.
- Simpler than callback-based asynchronous code.

---

# Java Concurrency Layers

## Layer 1: Thread Management

- Thread
- ExecutorService
- ForkJoinPool
- Virtual Threads

---

## Layer 2: Thread Safety

- synchronized
- Lock
- ReentrantLock
- volatile
- Atomic Classes

---

## Layer 3: High-Level Concurrency

- CompletableFuture
- ConcurrentHashMap
- BlockingQueue
- ReadWriteLock
- ForkJoin Framework

---

# Quick Summary

| Problem | Java Solution |
|----------|---------------|
| Run tasks concurrently | Thread, ExecutorService |
| Protect shared state | synchronized, Lock |
| Memory visibility | volatile |
| Atomic updates | AtomicInteger |
| Reuse worker threads | Thread Pool |
| Async programming | CompletableFuture |
| Thread-safe collections | ConcurrentHashMap |
| Producer-consumer workflow | BlockingQueue |
| Read-heavy workloads | ReadWriteLock |
| Parallel computation | ForkJoinPool |
| Massive concurrency | Virtual Threads |

---

# Interview-Friendly Answer

> Java handles concurrency using threads, synchronization mechanisms (`synchronized`, `Lock`), memory visibility controls (`volatile`), atomic operations (`AtomicInteger`), and high-level concurrency utilities from `java.util.concurrent` such as `ExecutorService`, `CompletableFuture`, `ConcurrentHashMap`, and `BlockingQueue`. Modern Java (Java 21+) also introduces Virtual Threads, allowing millions of lightweight concurrent tasks while simplifying concurrent application development.
