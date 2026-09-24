## 1. Built-in / Default Comparators in Java

Java provides static factory methods inside `Comparator` and standard class methods for common operations:

| Method / Utility | Description | Example Usage |
| --- | --- | --- |
| **`Comparator.naturalOrder()`** | Uses the object's default `Comparable` order (e.g., numbers $1 \to 9$, strings $A \to Z$). | `Comparator.naturalOrder()` |
| **`Comparator.reverseOrder()`** | Reverses the natural ordering (descending). | `Comparator.reverseOrder()` |
| **`Comparator.comparing(keyExtractor)`** | Extracts a sorting key (e.g., getting an object field). | `Comparator.comparing(User::getAge)` |
| **`Comparator.nullsFirst(...)`** | Places `null` elements at the beginning. | `Comparator.nullsFirst(Comparator.naturalOrder())` |
| **`Comparator.nullsLast(...)`** | Places `null` elements at the end. | `Comparator.nullsLast(Comparator.naturalOrder())` |
| **`.thenComparing(...)`** | Chaining secondary sorting criteria (break ties). | `.thenComparing(User::getName)` |

---

## 2. How to Write a Custom Comparator

Suppose you have a `Person` class:

```java
class Person {
    String name;
    int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge() { return age; }

    @Override
    public String toString() { return name + " (" + age + ")"; }
}

```

### Options to create a `Comparator<Person>`:

```java
// 1. Lambda Expression (Age Ascending)
Comparator<Person> byAge = (p1, p2) -> Integer.compare(p1.getAge(), p2.getAge());

// 2. Method Reference (Age Descending)
Comparator<Person> byAgeDesc = Comparator.comparingInt(Person::getAge).reversed();

// 3. Multi-field Chaining (Age Ascending, then Name Ascending)
Comparator<Person> byAgeThenName = Comparator.comparing(Person::getAge)
                                             .thenComparing(Person::getName);

```

---

## 3. Applying Comparators Across Java Structures

### A. Arrays (`Arrays.sort`)

Sorts an object array in place using the custom comparator.

```java
Person[] peopleArr = { new Person("Alice", 30), new Person("Bob", 25) };

Arrays.sort(peopleArr, Comparator.comparing(Person::getAge));
// Result: [Bob (25), Alice (30)]

```

### B. PriorityQueue (Min-Heap / Max-Heap)

Pass the comparator directly into the constructor to set priority order.

```java
// Max-Heap (Highest age served first)
PriorityQueue<Person> maxHeap = new PriorityQueue<>(Comparator.comparingInt(Person::getAge).reversed());

maxHeap.add(new Person("Alice", 30));
maxHeap.add(new Person("Bob", 25));

System.out.println(maxHeap.poll()); // Prints: Alice (30)

```

### C. TreeSet (Sorted Set)

Maintains automatic sorted order on insertion based on the comparator.

```java
TreeSet<Person> personSet = new TreeSet<>(Comparator.comparing(Person::getName));

personSet.add(new Person("Bob", 25));
personSet.add(new Person("Alice", 30));

System.out.println(personSet); // Prints: [Alice (30), Bob (25)]

```

### D. TreeMap (Sorted Key-Value Map)

Sorts entries **by Key** dynamically using the provided comparator.

```java
// TreeMap ordered by Person key's age ascending
TreeMap<Person, String> personMap = new TreeMap<>(Comparator.comparingInt(Person::getAge));

personMap.put(new Person("Alice", 30), "Engineer");
personMap.put(new Person("Bob", 25), "Doctor");

// Keys printed in age order: Bob (25), Alice (30)
personMap.forEach((key, val) -> System.out.println(key + " -> " + val));

```

### E. Collections (`List.sort` / `Collections.sort`)

```java
List<Person> personList = new ArrayList<>(List.of(
    new Person("Alice", 30),
    new Person("Bob", 25)
));

// Direct list method (Java 8+)
personList.sort(Comparator.comparing(Person::getAge));

```

---

## Key Takeaway

For collections backed by balanced trees (`TreeSet`, `TreeMap`) or heaps (`PriorityQueue`), the **Comparator must be passed at initialization** via the constructor, as it defines structural insertion behavior. For linear structures (`Arrays`, `List`), the **Comparator is passed directly to the sorting method**.
