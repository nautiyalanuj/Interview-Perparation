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


