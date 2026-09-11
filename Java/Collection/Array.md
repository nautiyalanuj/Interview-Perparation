- Declaration 

```
int[] arr = new int[5]; // [0,0,0,0,0] 

int[] arr = {1, 2, 3, 4, 5}; 

String[] names = {"Anuj", "John", "Alex"}; 
```
- 2D array declaration

```
int[][] matrix = new int[3][4];

int[][] matrix = { 

{1, 2}, 

{3, 4}, 

{5, 6} 

}; 
```
- Jagged Arrays
```
int[][] arr = {
  {1, 2},
  {3, 4, 5},
  {6}
};


int[][] arr = new int[3][];

arr[0] = new int[2];
arr[1] = new int[5];
arr[2] = new int[1];
```

- 2D Traversal
```
for (int i = 0; i < matrix.length; i++) {
  for (int j = 0; j < matrix[i].length; j++) {
    System.out.print(matrix[i][j] + " ");
  }
}


for (int[] row : matrix) {
  for (int num : row) {
  System.out.print(num + " ");
}
System.out.println();
}
```
- Length
  - arr.length
- Sort
  - Arrays.sort(arr);
  - Arrays.sort(arr, Collections.reverseOrder()); //Descending Order 
- Convert String to char array
  - s.toCharArray()
- Convert Array to String
  - Arrays.toString(arr) 
  - Arrays.deepToString(matrix) //2D array
- Compare
  - Arrays.equals(arr1, arr2);
  - Arrays.deepEquals(matrix1, matrix2); 
- Search in Array
  - Arrays.binarySearch(arr, 5); 
- Fill Array
  - Arrays.fill(arr, -1); 
- Copy Arrays 
```
int[] arr = {1, 2, 3}; 
int[] copy = Arrays.copyOf(arr, arr.length); 
```

- Partial Copy 
```
int[] sub = Arrays.copyOfRange(arr, 1, 3); 
```

- Return int array
  - return new int[]{nums[i], nums[i]}; 

- Character array to string 

```
char[] alphabet = {'h', 'e', 'l', 'l', 'o'};  
String result = String.valueOf(alphabet); 
```
