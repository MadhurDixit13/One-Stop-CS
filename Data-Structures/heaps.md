# Heap Data Structure

## Overview
A heap is a complete binary tree that satisfies the heap property. It's commonly used to implement priority queues and for efficient sorting algorithms. Heaps come in two main types: min-heap and max-heap.

## Types of Heaps

### Min-Heap
- **Property**: Parent node is always smaller than or equal to its children
- **Root**: Contains the minimum element
- **Use Cases**: Priority queues, Dijkstra's algorithm

### Max-Heap
- **Property**: Parent node is always larger than or equal to its children
- **Root**: Contains the maximum element
- **Use Cases**: Heap sort, finding maximum elements

## Heap Properties

### Complete Binary Tree
- All levels are filled except possibly the last level
- Last level is filled from left to right
- Height: O(log n) where n is the number of nodes

### Heap Property
```
Min-Heap: parent ≤ children
Max-Heap: parent ≥ children
```

## Array Representation

### Index Mapping
For a node at index `i`:
- **Parent**: `(i - 1) // 2`
- **Left Child**: `2i + 1`
- **Right Child**: `2i + 2`

### Example
```
Array: [10, 20, 15, 30, 40]
Heap Structure:
        10
       /  \
     20    15
    /  \
  30    40

Index:  0   1   2   3   4
Value: 10  20  15  30  40
```

## Heap Implementation

### Basic Heap Class
```python
class MinHeap:
    def __init__(self):
        self.heap = []
    
    def parent(self, i):
        """Get parent index of node at index i."""
        return (i - 1) // 2
    
    def left_child(self, i):
        """Get left child index of node at index i."""
        return 2 * i + 1
    
    def right_child(self, i):
        """Get right child index of node at index i."""
        return 2 * i + 2
    
    def swap(self, i, j):
        """Swap elements at indices i and j."""
        self.heap[i], self.heap[j] = self.heap[j], self.heap[i]
    
    def insert(self, key):
        """Insert a new key into the heap."""
        # Add the new key to the end
        self.heap.append(key)
        
        # Fix the heap property
        self._heapify_up(len(self.heap) - 1)
    
    def _heapify_up(self, index):
        """Move element up to maintain heap property."""
        while index > 0:
            parent_index = self.parent(index)
            if self.heap[index] >= self.heap[parent_index]:
                break
            self.swap(index, parent_index)
            index = parent_index
    
    def extract_min(self):
        """Extract minimum element from heap."""
        if len(self.heap) == 0:
            raise IndexError("Heap is empty")
        
        if len(self.heap) == 1:
            return self.heap.pop()
        
        # Store the minimum value
        min_val = self.heap[0]
        
        # Move last element to root
        self.heap[0] = self.heap.pop()
        
        # Fix the heap property
        self._heapify_down(0)
        
        return min_val
    
    def _heapify_down(self, index):
        """Move element down to maintain heap property."""
        while True:
            smallest = index
            left = self.left_child(index)
            right = self.right_child(index)
            
            # Find the smallest among node and its children
            if left < len(self.heap) and self.heap[left] < self.heap[smallest]:
                smallest = left
            
            if right < len(self.heap) and self.heap[right] < self.heap[smallest]:
                smallest = right
            
            # If the smallest is the node itself, we're done
            if smallest == index:
                break
            
            # Otherwise, swap and continue
            self.swap(index, smallest)
            index = smallest
    
    def peek(self):
        """Get minimum element without removing it."""
        if len(self.heap) == 0:
            raise IndexError("Heap is empty")
        return self.heap[0]
    
    def size(self):
        """Get number of elements in heap."""
        return len(self.heap)
    
    def is_empty(self):
        """Check if heap is empty."""
        return len(self.heap) == 0
    
    def __str__(self):
        return str(self.heap)

# Example usage
heap = MinHeap()
heap.insert(10)
heap.insert(20)
heap.insert(15)
heap.insert(30)
heap.insert(40)
print(heap)  # [10, 20, 15, 30, 40]
print(heap.extract_min())  # 10
print(heap)  # [15, 20, 40, 30]
```

## Max-Heap Implementation

```python
class MaxHeap:
    def __init__(self):
        self.heap = []
    
    def parent(self, i):
        return (i - 1) // 2
    
    def left_child(self, i):
        return 2 * i + 1
    
    def right_child(self, i):
        return 2 * i + 2
    
    def swap(self, i, j):
        self.heap[i], self.heap[j] = self.heap[j], self.heap[i]
    
    def insert(self, key):
        self.heap.append(key)
        self._heapify_up(len(self.heap) - 1)
    
    def _heapify_up(self, index):
        while index > 0:
            parent_index = self.parent(index)
            if self.heap[index] <= self.heap[parent_index]:
                break
            self.swap(index, parent_index)
            index = parent_index
    
    def extract_max(self):
        if len(self.heap) == 0:
            raise IndexError("Heap is empty")
        
        if len(self.heap) == 1:
            return self.heap.pop()
        
        max_val = self.heap[0]
        self.heap[0] = self.heap.pop()
        self._heapify_down(0)
        
        return max_val
    
    def _heapify_down(self, index):
        while True:
            largest = index
            left = self.left_child(index)
            right = self.right_child(index)
            
            if left < len(self.heap) and self.heap[left] > self.heap[largest]:
                largest = left
            
            if right < len(self.heap) and self.heap[right] > self.heap[largest]:
                largest = right
            
            if largest == index:
                break
            
            self.swap(index, largest)
            index = largest
    
    def peek(self):
        if len(self.heap) == 0:
            raise IndexError("Heap is empty")
        return self.heap[0]
    
    def size(self):
        return len(self.heap)
    
    def is_empty(self):
        return len(self.heap) == 0
```

## Heap Operations

### Build Heap from Array
```python
def build_heap(arr, is_max_heap=False):
    """Build heap from array in O(n) time."""
    n = len(arr)
    
    # Start from the last non-leaf node
    start_index = (n // 2) - 1
    
    for i in range(start_index, -1, -1):
        if is_max_heap:
            heapify_max(arr, n, i)
        else:
            heapify_min(arr, n, i)
    
    return arr

def heapify_min(arr, n, i):
    """Heapify subtree rooted at index i for min-heap."""
    smallest = i
    left = 2 * i + 1
    right = 2 * i + 2
    
    if left < n and arr[left] < arr[smallest]:
        smallest = left
    
    if right < n and arr[right] < arr[smallest]:
        smallest = right
    
    if smallest != i:
        arr[i], arr[smallest] = arr[smallest], arr[i]
        heapify_min(arr, n, smallest)

def heapify_max(arr, n, i):
    """Heapify subtree rooted at index i for max-heap."""
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2
    
    if left < n and arr[left] > arr[largest]:
        largest = left
    
    if right < n and arr[right] > arr[largest]:
        largest = right
    
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify_max(arr, n, largest)

# Example
arr = [4, 10, 3, 5, 1]
build_heap(arr, is_max_heap=True)
print(arr)  # [10, 5, 3, 4, 1]
```

### Decrease/Increase Key
```python
def decrease_key_min_heap(heap, index, new_value):
    """Decrease key value at index in min-heap."""
    if new_value > heap[index]:
        raise ValueError("New value is greater than current value")
    
    heap[index] = new_value
    
    # Heapify up
    while index > 0:
        parent_index = (index - 1) // 2
        if heap[index] >= heap[parent_index]:
            break
        heap[index], heap[parent_index] = heap[parent_index], heap[index]
        index = parent_index

def increase_key_max_heap(heap, index, new_value):
    """Increase key value at index in max-heap."""
    if new_value < heap[index]:
        raise ValueError("New value is less than current value")
    
    heap[index] = new_value
    
    # Heapify up
    while index > 0:
        parent_index = (index - 1) // 2
        if heap[index] <= heap[parent_index]:
            break
        heap[index], heap[parent_index] = heap[parent_index], heap[index]
        index = parent_index
```

### Delete Key
```python
def delete_key_min_heap(heap, index):
    """Delete key at index from min-heap."""
    # Decrease key to negative infinity
    decrease_key_min_heap(heap, index, float('-inf'))
    
    # Extract minimum
    return extract_min(heap)
```

## Heap Sort

```python
def heap_sort(arr):
    """Sort array using heap sort algorithm."""
    n = len(arr)
    
    # Build max heap
    for i in range(n // 2 - 1, -1, -1):
        heapify_max(arr, n, i)
    
    # Extract elements one by one
    for i in range(n - 1, 0, -1):
        # Move current root to end
        arr[0], arr[i] = arr[i], arr[0]
        
        # Call heapify on the reduced heap
        heapify_max(arr, i, 0)
    
    return arr

# Time Complexity: O(n log n)
# Space Complexity: O(1)
```

## Priority Queue Implementation

```python
class PriorityQueue:
    def __init__(self):
        self.heap = []
        self.index = 0
    
    def push(self, item, priority):
        """Add item with priority to queue."""
        heapq.heappush(self.heap, (priority, self.index, item))
        self.index += 1
    
    def pop(self):
        """Remove and return highest priority item."""
        if self.is_empty():
            raise IndexError("Queue is empty")
        priority, index, item = heapq.heappop(self.heap)
        return item
    
    def peek(self):
        """Get highest priority item without removing."""
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self.heap[0][2]
    
    def is_empty(self):
        return len(self.heap) == 0
    
    def size(self):
        return len(self.heap)

# Example usage
pq = PriorityQueue()
pq.push("task1", 3)
pq.push("task2", 1)
pq.push("task3", 2)
print(pq.pop())  # task2 (priority 1)
print(pq.pop())  # task3 (priority 2)
```

## Advanced Heap Applications

### Kth Largest/Smallest Element
```python
def find_kth_largest(nums, k):
    """Find kth largest element using min-heap."""
    heap = []
    
    for num in nums:
        if len(heap) < k:
            heapq.heappush(heap, num)
        elif num > heap[0]:
            heapq.heapreplace(heap, num)
    
    return heap[0]

def find_kth_smallest(nums, k):
    """Find kth smallest element using max-heap."""
    heap = []
    
    for num in nums:
        if len(heap) < k:
            heapq.heappush(heap, -num)  # Use negative for max-heap
        elif num < -heap[0]:
            heapq.heapreplace(heap, -num)
    
    return -heap[0]

# Time Complexity: O(n log k)
# Space Complexity: O(k)
```

### Merge K Sorted Lists
```python
def merge_k_sorted_lists(lists):
    """Merge k sorted linked lists using heap."""
    import heapq
    
    heap = []
    result = []
    
    # Initialize heap with first element from each list
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst[0], i, 0))
    
    # Merge lists
    while heap:
        val, list_idx, element_idx = heapq.heappop(heap)
        result.append(val)
        
        # Add next element from the same list
        if element_idx + 1 < len(lists[list_idx]):
            heapq.heappush(heap, (lists[list_idx][element_idx + 1], list_idx, element_idx + 1))
    
    return result

# Time Complexity: O(n log k) where n is total elements
# Space Complexity: O(k)
```

### Running Median
```python
class RunningMedian:
    def __init__(self):
        self.small = []  # max-heap (use negative values)
        self.large = []  # min-heap
    
    def add_num(self, num):
        """Add number and maintain median."""
        # Add to appropriate heap
        if not self.small or num <= -self.small[0]:
            heapq.heappush(self.small, -num)
        else:
            heapq.heappush(self.large, num)
        
        # Balance heaps
        if len(self.small) > len(self.large) + 1:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        elif len(self.large) > len(self.small) + 1:
            val = heapq.heappop(self.large)
            heapq.heappush(self.small, -val)
    
    def find_median(self):
        """Find current median."""
        if len(self.small) > len(self.large):
            return -self.small[0]
        elif len(self.large) > len(self.small):
            return self.large[0]
        else:
            return (-self.small[0] + self.large[0]) / 2
```

## Performance Analysis

### Time Complexity

| Operation | Time Complexity |
|-----------|----------------|
| Insert | O(log n) |
| Extract Min/Max | O(log n) |
| Peek | O(1) |
| Build Heap | O(n) |
| Heap Sort | O(n log n) |
| Decrease/Increase Key | O(log n) |
| Delete | O(log n) |

### Space Complexity
- **Heap Storage**: O(n)
- **Operations**: O(1) for most operations, O(log n) for recursive heapify

## Applications

### Real-world Uses
1. **Priority Queues**: Task scheduling, process management
2. **Graph Algorithms**: Dijkstra's shortest path, Prim's MST
3. **Sorting**: Heap sort algorithm
4. **Statistics**: Finding kth largest/smallest elements
5. **Memory Management**: Garbage collection, memory allocation
6. **Data Compression**: Huffman coding
7. **Operating Systems**: Process scheduling

### When to Use Heaps
- **Pros**:
  - Efficient insertion and extraction
  - O(log n) operations
  - Space efficient
  - Good for priority-based operations

- **Cons**:
  - No efficient search for arbitrary elements
  - Not good for sorted traversal
  - Cache performance can be poor

### Alternatives
- **Balanced BST**: When you need search operations
- **Array**: When you only need min/max operations
- **Linked List**: When insertion order matters

## Best Practices

1. **Use Built-in Libraries**: Python's `heapq` module for production code
2. **Index Management**: Be careful with index calculations in custom implementations
3. **Memory Considerations**: Heaps can cause memory fragmentation
4. **Thread Safety**: Consider thread safety for concurrent applications
5. **Performance Tuning**: Profile heap operations in performance-critical code

Heaps are essential data structures for priority-based operations and form the foundation for many advanced algorithms in computer science.
