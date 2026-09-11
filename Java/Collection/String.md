- Equals
  - str1.equals(str2)
  - str1.equalsIgnoreCase(str2)
- Length
  - length()
- Concatenation
  - String r1 = a + " " + b;
  - String r2 = a.concat(" ").concat(b);
  - String r3 = String.join(", ", "x", "y", "z");  // "x, y, z"
- Reverse
  -  sb.reverse()
- Traversal 

```
for (int i = 0; i < s.length(); i++) { 
    char c = s.charAt(i); 
} 

for (char c : s.toCharArray()) { ... } 
```
- StringBuilder
  - append // to add at end //O(1)
  - sb.insert(0, "hello "); //O(n) insert at start
 
- Character to string
```
char ch = 'A';
String str = String.valueOf(ch); // Most efficient
// or
String str2 = Character.toString(ch);   
```

- Substring
```
String s = "Hello World";  
s.substring(6);          // "World"      — from index to end 
s.substring(0, 5);       // "Hello"      — [start, end) end-exclusive 
s.indexOf("World");      // 6 
s.lastIndexOf("l");      // 9 
s.contains("lo W");      // true 
s.startsWith("Hello");   // true 
s.endsWith("ld");        // true 
s.replace("World","Java"); 
s.strip();               // Java 11+, Unicode-aware (better than trim()) 
s.toUpperCase(); 
s.isBlank();             // Java 11+ 
```

- Split 
```
String csv = "a,b,c"; 
String[] parts = csv.split(",");            // ["a","b","c"] 

// limit parameter 
"a,b,c,d".split(",", 2);                    // ["a", "b,c,d"] 
"a,b,,,".split(",");                        // ["a","b"]  — trailing empties dropped 
"a,b,,,".split(",", -1);                    // ["a","b","","",""] — keep all 
  
// split takes a REGEX, not a literal 
"a.b.c".split(".");        // [] — '.' matches everything! 
"a.b.c".split("\\.");      // ["a","b","c"] ✓ 
"a|b".split("\\|");        // escape regex metacharacters 

  
// multiple delimiters 
"a,b;c d".split("[,; ]");                   // ["a","b","c","d"] 
"a1b22c".split("\\d+");                     // ["a","b","c"] 
```

- Character array to string 

char[] alphabet = {'h', 'e', 'l', 'l', 'o'};  
String result = String.valueOf(alphabet); // hello 

 

 
