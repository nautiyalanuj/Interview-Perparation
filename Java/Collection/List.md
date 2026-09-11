# ArrayList 
- An ArrayList is backed by a dynamically resizing array. It is the go-to implementation for most scenarios because it offers fast O(1) search and access times. 
- List<String> fruits = new ArrayList<>(); 
- fruits.add("Apple"); 
- fruits.get(0); 
- fruits.set(1, "Blueberry"); 
- fruits.remove(2); 
- fruits.remove("Apple"); 
- fruits.contains("Blueberry"); 

# LinkedList 
- Doubly-linked list, no single linked list in Java
- Frequent insertions or deletions from the middle, beginning, or end. 
- Fast modifications anywhere 
- Slow random access (must traverse elements) 

# Immutable List (Java 9+) 
- Great for read-only fixed data. You cannot add or remove elements. 
- List<String> fixedList = List.of("One", "Two", "Three"); 

# Fixed-Size List 

- Backed by an array. You can modify existing elements, but you cannot change the list's size. 
- List<String> fixedSize = Arrays.asList("Red", "Green", "Blue"); 

```
Iterate 
for (String fruit : fruits) { System.out.println(fruit); } 
Lambda => fruits.forEach(fruit -> System.out.println(fruit));
```
