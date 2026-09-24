- Set<String> set = new HashSet<>(List.of("A", "B", "C"));
- Traversal
  ```
  for (var item : set) {  // item is inferred as String
    System.out.println(item);
  }
  ```
  - Add
    - ```set.add("A");```
  - Remove
    - ```boolean removed = set.remove(element); // true if found & removed, false otherwise```
  - Contains
    - ```boolean exists = set.contains(element);  // O(1) for HashSet, O(log n) for TreeSet```
  - 
