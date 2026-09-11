- Same is default used for queue, linkedlist and stack.
- Same is developed as resizable, circular array
- Cache Locality (The Biggest Performance Factor)
  - Modern CPUs rely heavily on L1/L2/L3 caches. When you access an element in an array (ArrayDeque), the CPU automatically loads adjacent elements into the ultra-fast cache. 


|Operation | ArrayDeque| LinkedList | Why They Differ|
|-|-|-|-|
|Insert/Remove at Head| O(1)| amortizedO(1)| ArrayDeque is faster in practice because it just alters pointers. LinkedList must allocate a new node object on every insert.|
|Insert/Remove at Tail| O(1)| amortizedO(1)| ArrayDeque occasionally faces a resize penalty (array copy), but it is still faster on average due to cache efficiency.|
|Iteration (Traversing)| O(n)| O(n)| ArrayDeque loops over contiguous memory sequentially. LinkedList must follow random memory pointers.|
|Arbitrary Insertion| O(n)| O(n) | ArrayDeque shifts elements. LinkedList must traverse to the index before altering node pointers.|



| Feature| ArrayDeque |LinkedList|
|-|-|-|
|Underlying Data Structure|Circular Array |Doubly-Linked List|
|Implemented Interfaces|Deque, Queue| List, Deque, Queue|
|Memory Allocation| Contiguous blocks| Scattered nodes|
|Memory Overhead| Minimal (Raw array references) | Heavy (24-32 bytes per node wrapper)|
|Cache Locality| Excellent (High CPU cache hits)| Poor (High pointer chasing / cache misses)|
|Allows Null Elements?| ❌ No (Throws NullPointerException)| Yes|
|Index-based Access | (get(i))❌ |No direct index methodsYes (But runs in O(n) time)|


## ArrayDeque vs ArrayList
- The Structural Difference: ArrayList grows by appending items to the end of a contiguous array. If you insert an item at index 0, it must shift every single existing element one slot to the right (O(n)). ArrayDeque uses a circular layout with head and tail pointers. When you insert at the front, it simply moves the head pointer backward into empty array space, achieving true O(1) amortized insertion at both ends.
- The Rule of Thumb: Use ArrayList if you need to fetch elements by index (list.get(5)). Use ArrayDeque if you are only modifying the extreme ends of the collection.

|Feature / Class| ArrayDeque| ArrayList| LinkedList| Stack (Legacy)|
|-|-|-|-|-|
|Underlying Structure|Circular Array|Dynamic Resizable Array|Doubly-Linked List|Dynamic Array (Vector)|
|Core Design Purpose|Fast Deque / Stack / Queue|Fast Index Access / Lists|Sequential List / Deque|Legacy LIFO Stack|
|Thread Safety| ❌ Not Safe| ❌ Not Safe| ❌ Not Safe|Thread-Safe (Synchronized)|
|Allows null?| ❌ No| Yes| Yes| Yes|
|Memory Efficiency| Excellent| Excellent|❌ Poor (Node wrappers)| Good|
|Cache Locality| High| High | (Perfect)| ❌ Low (Pointer chasing)| High|


| Operation| ArrayDeque| ArrayList| LinkedList| Stack|
|-|-|-|-|-|
|Insert at Head| O(1) amortized| O(n) (Shifts all elements)| O(1)| O(n) (Shifts elements)|
|Insert at Tail| O(1) amortized| O(1) amortized| O(1)| O(1) amortized|
|Remove from Head| O(1)| O(n) (Shifts all elements)| O(1)| O(n) (Shifts elements)|
|Remove from Tail| O(1)| O(1)| O(1)| O(1)|
|Random Access (get(i))| ❌ N/A| O(1) (Direct index)| O(n) (Must traverse)| O(1)|


- Strictly speaking, random access is computationally possible in an ArrayDeque in O(1) time because the underlying data structure is a standard Java array.However, random access is not allowed because the Java architects intentionally left index-based methods (get(i), set(i, el)) out of the interface design.
- ArrayDeque implements the Deque (Double-Ended Queue) interface, not the List interface.
  - The pure definition of a Stack or Queue is to restrict access exclusively to the endpoints (the front and the back).
  - Adding methods like get(index) would violate the core semantic contract of what a Queue or Deque is supposed to do. If you need a structural contract that guarantees random access, Java provides the List interface (implemented by ArrayList).
