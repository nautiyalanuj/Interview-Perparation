# ArrayList 
- An ArrayList is backed by a dynamically resizing array. It is the go-to implementation for most scenarios because it offers fast O(1) search and access times. 
- List<String> fruits = new ArrayList<>(List.of(1, 2, 3)); 
- fruits.add("Apple"); 
- fruits.get(0); 
- fruits.set(1, "Blueberry"); 
- fruits.remove(2); // O(n)  
- fruits.remove("Apple"); // O(n) 
- fruits.contains("Blueberry"); // O(n) 
- int index = Collections.binarySearch(sortedList, target);
  - int index = Collections.binarySearch(list, key, [comparator](https://github.com/nautiyalanuj/Interview-Perparation/blob/main/Java/Collection/Comparator.md));

```
  class Fruit {
    String name;
    int price;
  }
  Comparator<Fruit> priceComparator = Comparator.comparingInt(f -> f.price);
  ```
- size() // Length 
- Iterate
    ```
    for (String fruit : fruits) { System.out.println(fruit); }
    Lambda => fruits.forEach(fruit -> System.out.println(fruit));
    ```
- LinkedList 
  - [ArrayDeque](https://github.com/nautiyalanuj/Interview-Perparation/blob/main/Java/Collection/ArrayDeque.md)

- Immutable List (Java 9+) : Great for read-only fixed data. You cannot add or remove elements. 
  - List<String> fixedList = List.of("One", "Two", "Three"); 

- Fixed-Size List : Backed by an array. You can modify existing elements, but you cannot change the list's size. 
  - List<String> fixedSize = Arrays.asList("Red", "Green", "Blue"); 

# Array vs ArrayList

### When to Use Array vs. ArrayList

While `ArrayList` is the default choice for general development due to its flexibility, plain arrays (`T[]` or primitive arrays like `int[]`) are preferred in specific scenarios:

#### Use Plain Arrays (`T[]` / `primitive[]`) when:

1. **Working with Primitives:** Arrays store raw primitives (`int`, `double`, `boolean`) without performance overhead. `ArrayList<Integer>` requires wrapper objects (`Integer`), which causes automatic memory boxing/unboxing and higher memory footprint.
2. **Fixed Size & Memory Efficiency:** When size is known in advance and won't change, arrays eliminate the extra object overhead of `ArrayList` (capacity headers, growth buffers, modCount fields).
3. **Multi-Dimensional Data:** Matrix or grid structures (e.g., `int[][] grid`) are simpler and faster to access with arrays than `List<List<Integer>>`.
4. **Performance-Critical Code:** High-frequency loops, game engine code, low-latency trading, or low-level algorithms benefit from direct memory layout and minimal abstraction overhead.

---

### Conversion: Array to ArrayList (and vice versa)

#### 1. Array $\rightarrow$ ArrayList

* **For Object Arrays (`String[]`, `Integer[]`, etc.):**
```java
String[] fruitArray = {"Apple", "Banana", "Cherry"};

// Modifiable ArrayList (Most common)
List<String> fruitList = new ArrayList<>(Arrays.asList(fruitArray));

// Immutable List (Java 10+)
List<String> immutableList = List.of(fruitArray);

```


* **For Primitive Arrays (`int[]`, `double[]`, etc.):**
```java
int[] numbers = {1, 2, 3, 4, 5};

// Convert primitive array to List<Integer> using Streams
List<Integer> numberList = Arrays.stream(numbers)
                                 .boxed()
                                 .collect(Collectors.toList());

```



#### 2. ArrayList $\rightarrow$ Array

```java
List<String> fruitList = new ArrayList<>(List.of("Apple", "Banana"));

// Modern array allocation (Java 11+)
String[] fruitArray = fruitList.toArray(String[]::new);

```

---

### Key Advantages Comparison

| Metric / Feature | Plain Array (`T[]` / `int[]`) | `ArrayList<E>` |
| --- | --- | --- |
| **Resizing** | Fixed length upon creation. Cannot grow or shrink. | Dynamic size. Automatically grows by ~50% when full. |
| **Primitives Support** | Supports raw primitives directly (`int[]`, `byte[]`). | Requires object wrappers (`List<Integer>`), triggering boxing overhead. |
| **Type Safety & Generics** | Reified types (type checked at runtime). | Generic types erased at compile-time (`E`). |
| **Built-in Operations** | Basic array operations; utility functions require `Arrays` class. | Rich utility methods (`add`, `remove`, `contains`, `sort`, `subList`). |
| **Memory Footprint** | Low. Contiguous memory for elements only. | Higher. Object metadata + unused array capacity buffer. |
