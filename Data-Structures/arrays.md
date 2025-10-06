# Arrays Data Structure

## Overview
An array is a collection of elements stored in contiguous memory locations. Each element can be accessed directly by its index, making arrays one of the most fundamental and efficient data structures.

## Key Characteristics

### Properties
- **Fixed Size**: Arrays have a predetermined size
- **Homogeneous**: All elements are of the same data type
- **Contiguous Memory**: Elements are stored in adjacent memory locations
- **Zero-based Indexing**: First element is at index 0
- **Random Access**: O(1) time complexity for accessing any element

### Memory Layout
```
Index:    0    1    2    3    4    5
Value:   10   20   30   40   50   60
Memory: 1000 1004 1008 1012 1016 1020
```

## Array Operations

### Basic Operations

#### 1. Access (Read)
```python
def access_element(arr, index):
    """Access element at given index."""
    if 0 <= index < len(arr):
        return arr[index]
    else:
        raise IndexError("Index out of range")

# Time Complexity: O(1)
# Space Complexity: O(1)
```

#### 2. Search
```python
def linear_search(arr, target):
    """Linear search for target element."""
    for i in range(len(arr)):
        if arr[i] == target:
            return i
    return -1

# Time Complexity: O(n)
# Space Complexity: O(1)

def binary_search(arr, target):
    """Binary search for target element (requires sorted array)."""
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1

# Time Complexity: O(log n)
# Space Complexity: O(1)
```

#### 3. Insertion
```python
def insert_at_end(arr, element):
    """Insert element at the end of array."""
    arr.append(element)
    return arr

def insert_at_index(arr, index, element):
    """Insert element at specific index."""
    if 0 <= index <= len(arr):
        arr.insert(index, element)
        return arr
    else:
        raise IndexError("Index out of range")

def insert_sorted(arr, element):
    """Insert element in sorted position."""
    i = len(arr) - 1
    arr.append(element)  # Add space
    
    # Shift elements to make room
    while i >= 0 and arr[i] > element:
        arr[i + 1] = arr[i]
        i -= 1
    
    arr[i + 1] = element
    return arr

# Time Complexity: O(1) for end, O(n) for middle
# Space Complexity: O(1)
```

#### 4. Deletion
```python
def delete_by_value(arr, value):
    """Delete first occurrence of value."""
    try:
        index = arr.index(value)
        return arr.pop(index)
    except ValueError:
        return None

def delete_by_index(arr, index):
    """Delete element at specific index."""
    if 0 <= index < len(arr):
        return arr.pop(index)
    else:
        raise IndexError("Index out of range")

def delete_all_occurrences(arr, value):
    """Delete all occurrences of value."""
    return [x for x in arr if x != value]

# Time Complexity: O(n) for value deletion, O(1) for end deletion
# Space Complexity: O(1)
```

## Dynamic Arrays

### Implementation
```python
class DynamicArray:
    def __init__(self, capacity=4):
        self.capacity = capacity
        self.size = 0
        self.data = [None] * capacity
    
    def __len__(self):
        return self.size
    
    def __getitem__(self, index):
        if 0 <= index < self.size:
            return self.data[index]
        raise IndexError("Index out of range")
    
    def __setitem__(self, index, value):
        if 0 <= index < self.size:
            self.data[index] = value
        else:
            raise IndexError("Index out of range")
    
    def append(self, element):
        """Add element to end of array."""
        if self.size >= self.capacity:
            self._resize()
        
        self.data[self.size] = element
        self.size += 1
    
    def insert(self, index, element):
        """Insert element at specific index."""
        if index < 0 or index > self.size:
            raise IndexError("Index out of range")
        
        if self.size >= self.capacity:
            self._resize()
        
        # Shift elements to the right
        for i in range(self.size, index, -1):
            self.data[i] = self.data[i - 1]
        
        self.data[index] = element
        self.size += 1
    
    def delete(self, index):
        """Delete element at specific index."""
        if index < 0 or index >= self.size:
            raise IndexError("Index out of range")
        
        # Shift elements to the left
        for i in range(index, self.size - 1):
            self.data[i] = self.data[i + 1]
        
        self.size -= 1
    
    def _resize(self):
        """Double the capacity when array is full."""
        old_capacity = self.capacity
        self.capacity *= 2
        new_data = [None] * self.capacity
        
        for i in range(old_capacity):
            new_data[i] = self.data[i]
        
        self.data = new_data
    
    def __str__(self):
        return str([self.data[i] for i in range(self.size)])

# Example usage
arr = DynamicArray()
arr.append(1)
arr.append(2)
arr.append(3)
print(arr)  # [1, 2, 3]
arr.insert(1, 5)
print(arr)  # [1, 5, 2, 3]
arr.delete(2)
print(arr)  # [1, 5, 3]
```

## Multi-dimensional Arrays

### 2D Array Implementation
```python
class Matrix:
    def __init__(self, rows, cols, initial_value=0):
        self.rows = rows
        self.cols = cols
        self.data = [[initial_value for _ in range(cols)] for _ in range(rows)]
    
    def __getitem__(self, key):
        row, col = key
        return self.data[row][col]
    
    def __setitem__(self, key, value):
        row, col = key
        self.data[row][col] = value
    
    def transpose(self):
        """Return transpose of matrix."""
        result = Matrix(self.cols, self.rows)
        for i in range(self.rows):
            for j in range(self.cols):
                result[j, i] = self[i, j]
        return result
    
    def multiply(self, other):
        """Matrix multiplication."""
        if self.cols != other.rows:
            raise ValueError("Cannot multiply matrices")
        
        result = Matrix(self.rows, other.cols)
        for i in range(self.rows):
            for j in range(other.cols):
                for k in range(self.cols):
                    result[i, j] += self[i, k] * other[k, j]
        return result
    
    def __str__(self):
        return '\n'.join([' '.join(map(str, row)) for row in self.data])

# Example usage
m1 = Matrix(2, 3)
m1[0, 0] = 1; m1[0, 1] = 2; m1[0, 2] = 3
m1[1, 0] = 4; m1[1, 1] = 5; m1[1, 2] = 6
print("Matrix 1:")
print(m1)

m2 = m1.transpose()
print("\nTranspose:")
print(m2)
```

## Common Array Algorithms

### Sorting Algorithms

#### 1. Bubble Sort
```python
def bubble_sort(arr):
    """Bubble sort implementation."""
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        if not swapped:
            break
    return arr

# Time Complexity: O(n²) worst case, O(n) best case
# Space Complexity: O(1)
```

#### 2. Quick Sort
```python
def quick_sort(arr):
    """Quick sort implementation."""
    if len(arr) <= 1:
        return arr
    
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    
    return quick_sort(left) + middle + quick_sort(right)

# Time Complexity: O(n log n) average, O(n²) worst case
# Space Complexity: O(log n)
```

#### 3. Merge Sort
```python
def merge_sort(arr):
    """Merge sort implementation."""
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):
    """Merge two sorted arrays."""
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    return result

# Time Complexity: O(n log n)
# Space Complexity: O(n)
```

### Searching Algorithms

#### 1. Two Pointer Technique
```python
def two_sum_sorted(arr, target):
    """Find two numbers that sum to target in sorted array."""
    left, right = 0, len(arr) - 1
    
    while left < right:
        current_sum = arr[left] + arr[right]
        if current_sum == target:
            return [left, right]
        elif current_sum < target:
            left += 1
        else:
            right -= 1
    
    return []

def remove_duplicates(arr):
    """Remove duplicates from sorted array."""
    if not arr:
        return 0
    
    write_index = 1
    for read_index in range(1, len(arr)):
        if arr[read_index] != arr[read_index - 1]:
            arr[write_index] = arr[read_index]
            write_index += 1
    
    return write_index

# Time Complexity: O(n)
# Space Complexity: O(1)
```

#### 2. Sliding Window
```python
def max_sum_subarray(arr, k):
    """Find maximum sum of subarray of length k."""
    if len(arr) < k:
        return 0
    
    # Calculate sum of first window
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    # Slide the window
    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i - k] + arr[i]
        max_sum = max(max_sum, window_sum)
    
    return max_sum

def longest_substring_no_repeat(s):
    """Find length of longest substring without repeating characters."""
    char_set = set()
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        while s[right] in char_set:
            char_set.remove(s[left])
            left += 1
        
        char_set.add(s[right])
        max_length = max(max_length, right - left + 1)
    
    return max_length

# Time Complexity: O(n)
# Space Complexity: O(min(m, n)) where m is charset size
```

## Advanced Array Techniques

### Kadane's Algorithm
```python
def max_subarray_sum(arr):
    """Find maximum sum of contiguous subarray (Kadane's algorithm)."""
    max_ending_here = max_so_far = arr[0]
    
    for i in range(1, len(arr)):
        max_ending_here = max(arr[i], max_ending_here + arr[i])
        max_so_far = max(max_so_far, max_ending_here)
    
    return max_so_far

# Time Complexity: O(n)
# Space Complexity: O(1)
```

### Dutch National Flag Problem
```python
def sort_colors(arr):
    """Sort array of 0s, 1s, and 2s (Dutch National Flag problem)."""
    low = mid = 0
    high = len(arr) - 1
    
    while mid <= high:
        if arr[mid] == 0:
            arr[low], arr[mid] = arr[mid], arr[low]
            low += 1
            mid += 1
        elif arr[mid] == 1:
            mid += 1
        else:  # arr[mid] == 2
            arr[mid], arr[high] = arr[high], arr[mid]
            high -= 1
    
    return arr

# Time Complexity: O(n)
# Space Complexity: O(1)
```

### Stock Trading Problems
```python
def max_profit(prices):
    """Find maximum profit from buying and selling stock once."""
    if not prices:
        return 0
    
    min_price = prices[0]
    max_profit = 0
    
    for price in prices:
        min_price = min(min_price, price)
        max_profit = max(max_profit, price - min_price)
    
    return max_profit

def max_profit_multiple_transactions(prices):
    """Find maximum profit with multiple transactions allowed."""
    if not prices:
        return 0
    
    profit = 0
    for i in range(1, len(prices)):
        if prices[i] > prices[i - 1]:
            profit += prices[i] - prices[i - 1]
    
    return profit

# Time Complexity: O(n)
# Space Complexity: O(1)
```

## Performance Analysis

### Time Complexity Comparison

| Operation | Array | Dynamic Array |
|-----------|-------|---------------|
| Access | O(1) | O(1) |
| Search | O(n) | O(n) |
| Insert (end) | O(1) | O(1) amortized |
| Insert (middle) | O(n) | O(n) |
| Delete | O(n) | O(n) |
| Resize | N/A | O(n) |

### Space Complexity
- **Static Array**: O(n)
- **Dynamic Array**: O(n) with amortized O(1) append operations

## Applications

### Real-world Uses
1. **Image Processing**: Pixel arrays for image manipulation
2. **Audio Processing**: Sample arrays for audio signals
3. **Game Development**: Grid-based games, sprite sheets
4. **Scientific Computing**: Numerical computations, matrices
5. **Database Systems**: Row storage, index structures
6. **Cache Systems**: LRU cache implementation
7. **String Processing**: Character arrays for text manipulation

### When to Use Arrays
- **Pros**:
  - Fast random access (O(1))
  - Memory efficient (no extra pointers)
  - Cache-friendly (contiguous memory)
  - Simple implementation

- **Cons**:
  - Fixed size (unless dynamic)
  - Expensive insertion/deletion in middle
  - Memory waste if array is not full

### Alternatives
- **Linked Lists**: When frequent insertion/deletion needed
- **Hash Tables**: When key-value pairs are needed
- **Trees**: When hierarchical data is required
- **Dynamic Arrays**: When size is unknown initially

## Best Practices

1. **Memory Management**: Be aware of memory usage with large arrays
2. **Bounds Checking**: Always validate array indices
3. **Initialization**: Initialize arrays properly to avoid undefined behavior
4. **Resizing**: Use dynamic arrays when size is unknown
5. **Cache Optimization**: Access elements sequentially when possible
6. **Error Handling**: Handle edge cases (empty arrays, single elements)

Arrays remain one of the most fundamental and efficient data structures, forming the foundation for many other advanced data structures and algorithms.
