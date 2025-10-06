# Linked Lists Data Structure

## Overview
A linked list is a linear data structure where elements are stored in nodes, and each node points to the next node in the sequence. Unlike arrays, linked lists don't store elements in contiguous memory locations, providing dynamic memory allocation and efficient insertion/deletion operations.

## Types of Linked Lists

### 1. Singly Linked List
Each node contains data and a pointer to the next node.

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class SinglyLinkedList:
    def __init__(self):
        self.head = None
        self.size = 0
    
    def append(self, val):
        """Add element to the end of the list."""
        new_node = ListNode(val)
        
        if not self.head:
            self.head = new_node
        else:
            current = self.head
            while current.next:
                current = current.next
            current.next = new_node
        
        self.size += 1
    
    def prepend(self, val):
        """Add element to the beginning of the list."""
        new_node = ListNode(val)
        new_node.next = self.head
        self.head = new_node
        self.size += 1
    
    def insert(self, index, val):
        """Insert element at specific index."""
        if index < 0 or index > self.size:
            raise IndexError("Index out of range")
        
        if index == 0:
            self.prepend(val)
            return
        
        new_node = ListNode(val)
        current = self.head
        
        for _ in range(index - 1):
            current = current.next
        
        new_node.next = current.next
        current.next = new_node
        self.size += 1
    
    def delete(self, val):
        """Delete first occurrence of value."""
        if not self.head:
            return False
        
        if self.head.val == val:
            self.head = self.head.next
            self.size -= 1
            return True
        
        current = self.head
        while current.next:
            if current.next.val == val:
                current.next = current.next.next
                self.size -= 1
                return True
            current = current.next
        
        return False
    
    def delete_at_index(self, index):
        """Delete element at specific index."""
        if index < 0 or index >= self.size:
            raise IndexError("Index out of range")
        
        if index == 0:
            self.head = self.head.next
            self.size -= 1
            return
        
        current = self.head
        for _ in range(index - 1):
            current = current.next
        
        current.next = current.next.next
        self.size -= 1
    
    def search(self, val):
        """Search for value in the list."""
        current = self.head
        index = 0
        
        while current:
            if current.val == val:
                return index
            current = current.next
            index += 1
        
        return -1
    
    def get(self, index):
        """Get value at specific index."""
        if index < 0 or index >= self.size:
            raise IndexError("Index out of range")
        
        current = self.head
        for _ in range(index):
            current = current.next
        
        return current.val
    
    def reverse(self):
        """Reverse the linked list."""
        prev = None
        current = self.head
        
        while current:
            next_node = current.next
            current.next = prev
            prev = current
            current = next_node
        
        self.head = prev
    
    def to_list(self):
        """Convert linked list to Python list."""
        result = []
        current = self.head
        while current:
            result.append(current.val)
            current = current.next
        return result
    
    def __str__(self):
        return " -> ".join(map(str, self.to_list())) + " -> None"

# Example usage
ll = SinglyLinkedList()
ll.append(1)
ll.append(2)
ll.append(3)
ll.prepend(0)
print(ll)  # 0 -> 1 -> 2 -> 3 -> None
ll.reverse()
print(ll)  # 3 -> 2 -> 1 -> 0 -> None
```

### 2. Doubly Linked List
Each node contains data, pointer to next node, and pointer to previous node.

```python
class DoublyListNode:
    def __init__(self, val=0, prev=None, next=None):
        self.val = val
        self.prev = prev
        self.next = next

class DoublyLinkedList:
    def __init__(self):
        self.head = None
        self.tail = None
        self.size = 0
    
    def append(self, val):
        """Add element to the end of the list."""
        new_node = DoublyListNode(val)
        
        if not self.head:
            self.head = self.tail = new_node
        else:
            new_node.prev = self.tail
            self.tail.next = new_node
            self.tail = new_node
        
        self.size += 1
    
    def prepend(self, val):
        """Add element to the beginning of the list."""
        new_node = DoublyListNode(val)
        
        if not self.head:
            self.head = self.tail = new_node
        else:
            new_node.next = self.head
            self.head.prev = new_node
            self.head = new_node
        
        self.size += 1
    
    def insert(self, index, val):
        """Insert element at specific index."""
        if index < 0 or index > self.size:
            raise IndexError("Index out of range")
        
        if index == 0:
            self.prepend(val)
            return
        
        if index == self.size:
            self.append(val)
            return
        
        new_node = DoublyListNode(val)
        current = self.head
        
        for _ in range(index):
            current = current.next
        
        new_node.prev = current.prev
        new_node.next = current
        current.prev.next = new_node
        current.prev = new_node
        self.size += 1
    
    def delete(self, val):
        """Delete first occurrence of value."""
        current = self.head
        
        while current:
            if current.val == val:
                if current.prev:
                    current.prev.next = current.next
                else:
                    self.head = current.next
                
                if current.next:
                    current.next.prev = current.prev
                else:
                    self.tail = current.prev
                
                self.size -= 1
                return True
            
            current = current.next
        
        return False
    
    def delete_at_index(self, index):
        """Delete element at specific index."""
        if index < 0 or index >= self.size:
            raise IndexError("Index out of range")
        
        current = self.head
        for _ in range(index):
            current = current.next
        
        if current.prev:
            current.prev.next = current.next
        else:
            self.head = current.next
        
        if current.next:
            current.next.prev = current.prev
        else:
            self.tail = current.prev
        
        self.size -= 1
    
    def search(self, val):
        """Search for value in the list."""
        current = self.head
        index = 0
        
        while current:
            if current.val == val:
                return index
            current = current.next
            index += 1
        
        return -1
    
    def reverse(self):
        """Reverse the doubly linked list."""
        current = self.head
        
        while current:
            # Swap prev and next pointers
            current.prev, current.next = current.next, current.prev
            current = current.prev
        
        # Swap head and tail
        self.head, self.tail = self.tail, self.head
    
    def to_list(self):
        """Convert to Python list."""
        result = []
        current = self.head
        while current:
            result.append(current.val)
            current = current.next
        return result
    
    def to_list_reverse(self):
        """Convert to Python list in reverse order."""
        result = []
        current = self.tail
        while current:
            result.append(current.val)
            current = current.prev
        return result
    
    def __str__(self):
        return " <-> ".join(map(str, self.to_list()))
```

### 3. Circular Linked List
The last node points back to the first node.

```python
class CircularLinkedList:
    def __init__(self):
        self.head = None
        self.size = 0
    
    def append(self, val):
        """Add element to the end of circular list."""
        new_node = ListNode(val)
        
        if not self.head:
            self.head = new_node
            new_node.next = self.head
        else:
            current = self.head
            while current.next != self.head:
                current = current.next
            
            current.next = new_node
            new_node.next = self.head
        
        self.size += 1
    
    def prepend(self, val):
        """Add element to the beginning of circular list."""
        new_node = ListNode(val)
        
        if not self.head:
            self.head = new_node
            new_node.next = self.head
        else:
            current = self.head
            while current.next != self.head:
                current = current.next
            
            new_node.next = self.head
            self.head = new_node
            current.next = self.head
        
        self.size += 1
    
    def delete(self, val):
        """Delete first occurrence of value."""
        if not self.head:
            return False
        
        current = self.head
        prev = None
        
        # Find the node to delete
        while True:
            if current.val == val:
                break
            prev = current
            current = current.next
            
            # If we've gone full circle
            if current == self.head:
                return False
        
        # If only one node
        if current.next == self.head and current == self.head:
            self.head = None
            self.size -= 1
            return True
        
        # If deleting head
        if current == self.head:
            prev = self.head
            while prev.next != self.head:
                prev = prev.next
            
            self.head = current.next
            prev.next = self.head
        else:
            prev.next = current.next
        
        self.size -= 1
        return True
    
    def to_list(self):
        """Convert to Python list."""
        if not self.head:
            return []
        
        result = []
        current = self.head
        
        while True:
            result.append(current.val)
            current = current.next
            if current == self.head:
                break
        
        return result
    
    def __str__(self):
        if not self.head:
            return "Empty circular list"
        
        result = []
        current = self.head
        
        while True:
            result.append(str(current.val))
            current = current.next
            if current == self.head:
                break
        
        return " -> ".join(result) + " -> (back to head)"
```

## Common Linked List Algorithms

### Fast and Slow Pointer Technique
```python
def find_middle(head):
    """Find middle node using fast and slow pointers."""
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    return slow

def has_cycle(head):
    """Detect if linked list has a cycle."""
    if not head or not head.next:
        return False
    
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        
        if slow == fast:
            return True
    
    return False

def find_cycle_start(head):
    """Find the start of cycle in linked list."""
    if not head or not head.next:
        return None
    
    # First phase: detect cycle
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        
        if slow == fast:
            break
    else:
        return None  # No cycle
    
    # Second phase: find cycle start
    slow = head
    while slow != fast:
        slow = slow.next
        fast = fast.next
    
    return slow
```

### Merge and Sort Operations
```python
def merge_sorted_lists(l1, l2):
    """Merge two sorted linked lists."""
    dummy = ListNode(0)
    current = dummy
    
    while l1 and l2:
        if l1.val <= l2.val:
            current.next = l1
            l1 = l1.next
        else:
            current.next = l2
            l2 = l2.next
        current = current.next
    
    # Attach remaining nodes
    current.next = l1 or l2
    
    return dummy.next

def merge_sort_linked_list(head):
    """Sort linked list using merge sort."""
    if not head or not head.next:
        return head
    
    # Find middle
    slow = fast = head
    prev = None
    
    while fast and fast.next:
        prev = slow
        slow = slow.next
        fast = fast.next.next
    
    # Split the list
    prev.next = None
    
    # Recursively sort both halves
    left = merge_sort_linked_list(head)
    right = merge_sort_linked_list(slow)
    
    # Merge sorted halves
    return merge_sorted_lists(left, right)

def partition_list(head, x):
    """Partition list around value x."""
    before_head = before = ListNode(0)
    after_head = after = ListNode(0)
    
    while head:
        if head.val < x:
            before.next = head
            before = before.next
        else:
            after.next = head
            after = after.next
        head = head.next
    
    after.next = None
    before.next = after_head.next
    
    return before_head.next
```

### Advanced Operations
```python
def remove_nth_from_end(head, n):
    """Remove nth node from end of list."""
    dummy = ListNode(0)
    dummy.next = head
    first = second = dummy
    
    # Move first pointer n+1 steps ahead
    for _ in range(n + 1):
        first = first.next
    
    # Move both pointers until first reaches end
    while first:
        first = first.next
        second = second.next
    
    # Remove the nth node
    second.next = second.next.next
    
    return dummy.next

def rotate_list(head, k):
    """Rotate list to the right by k places."""
    if not head or not head.next or k == 0:
        return head
    
    # Find length and tail
    length = 1
    tail = head
    
    while tail.next:
        tail = tail.next
        length += 1
    
    # Adjust k
    k = k % length
    if k == 0:
        return head
    
    # Find new tail
    new_tail = head
    for _ in range(length - k - 1):
        new_tail = new_tail.next
    
    # Rotate
    new_head = new_tail.next
    new_tail.next = None
    tail.next = head
    
    return new_head

def swap_pairs(head):
    """Swap every two adjacent nodes."""
    dummy = ListNode(0)
    dummy.next = head
    prev = dummy
    
    while prev.next and prev.next.next:
        first = prev.next
        second = prev.next.next
        
        # Swap
        prev.next = second
        first.next = second.next
        second.next = first
        
        prev = first
    
    return dummy.next

def reverse_nodes_in_k_group(head, k):
    """Reverse nodes in groups of k."""
    def reverse_list(start, end):
        """Reverse list from start to end (exclusive)."""
        prev = end
        current = start
        
        while current != end:
            next_node = current.next
            current.next = prev
            prev = current
            current = next_node
        
        return prev
    
    dummy = ListNode(0)
    dummy.next = head
    prev_group_end = dummy
    
    while True:
        # Find kth node
        kth_node = prev_group_end
        for _ in range(k):
            kth_node = kth_node.next
            if not kth_node:
                return dummy.next
        
        # Store next group start
        next_group_start = kth_node.next
        
        # Reverse current group
        group_start = prev_group_end.next
        prev_group_end.next = reverse_list(group_start, next_group_start)
        
        # Move to next group
        prev_group_end = group_start
    
    return dummy.next
```

### Palindrome Detection
```python
def is_palindrome(head):
    """Check if linked list is palindrome."""
    if not head or not head.next:
        return True
    
    # Find middle
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    # Reverse second half
    second_half = reverse_linked_list(slow)
    
    # Compare first and second half
    first_half = head
    while second_half:
        if first_half.val != second_half.val:
            return False
        first_half = first_half.next
        second_half = second_half.next
    
    return True

def reverse_linked_list(head):
    """Reverse linked list."""
    prev = None
    current = head
    
    while current:
        next_node = current.next
        current.next = prev
        prev = current
        current = next_node
    
    return prev
```

### Intersection and Union
```python
def get_intersection_node(headA, headB):
    """Find intersection node of two linked lists."""
    if not headA or not headB:
        return None
    
    # Get lengths
    lenA = get_length(headA)
    lenB = get_length(headB)
    
    # Move longer list forward
    if lenA > lenB:
        for _ in range(lenA - lenB):
            headA = headA.next
    else:
        for _ in range(lenB - lenA):
            headB = headB.next
    
    # Find intersection
    while headA and headB:
        if headA == headB:
            return headA
        headA = headA.next
        headB = headB.next
    
    return None

def get_length(head):
    """Get length of linked list."""
    length = 0
    while head:
        length += 1
        head = head.next
    return length

def remove_duplicates(head):
    """Remove duplicates from sorted linked list."""
    if not head:
        return head
    
    current = head
    while current.next:
        if current.val == current.next.val:
            current.next = current.next.next
        else:
            current = current.next
    
    return head

def remove_duplicates_unsorted(head):
    """Remove duplicates from unsorted linked list."""
    if not head:
        return head
    
    seen = set()
    prev = None
    current = head
    
    while current:
        if current.val in seen:
            prev.next = current.next
        else:
            seen.add(current.val)
            prev = current
        current = current.next
    
    return head
```

## Performance Analysis

### Time Complexity

| Operation | Singly Linked List | Doubly Linked List | Circular Linked List |
|-----------|-------------------|-------------------|---------------------|
| Access | O(n) | O(n) | O(n) |
| Search | O(n) | O(n) | O(n) |
| Insert (beginning) | O(1) | O(1) | O(1) |
| Insert (end) | O(n) | O(1) | O(n) |
| Insert (middle) | O(n) | O(n) | O(n) |
| Delete (beginning) | O(1) | O(1) | O(1) |
| Delete (end) | O(n) | O(1) | O(n) |
| Delete (middle) | O(n) | O(n) | O(n) |

### Space Complexity
- **Storage**: O(n) for n elements
- **Operations**: O(1) for most operations (except recursive algorithms)

## Applications

### Real-world Uses
1. **Music Players**: Playlist navigation
2. **Browser History**: Back/forward navigation
3. **Undo Operations**: Text editors, graphics software
4. **Hash Tables**: Collision resolution with chaining
5. **Graph Representation**: Adjacency lists
6. **Memory Management**: Free list management
7. **Polynomial Representation**: Sparse polynomials

### When to Use Linked Lists
- **Pros**:
  - Dynamic size
  - Efficient insertion/deletion at beginning
  - No memory waste
  - Easy to implement

- **Cons**:
  - No random access
  - Extra memory for pointers
  - Poor cache performance
  - Not suitable for binary search

### Alternatives
- **Arrays**: When random access is needed
- **Dynamic Arrays**: When size is unknown
- **Hash Tables**: When fast search is needed
- **Trees**: When hierarchical data is required

## Best Practices

1. **Memory Management**: Always handle edge cases (empty list, single node)
2. **Pointer Safety**: Check for null pointers before dereferencing
3. **Cycle Detection**: Be careful with circular references
4. **Dummy Nodes**: Use dummy nodes to simplify edge cases
5. **Iterative vs Recursive**: Consider stack overflow for large lists
6. **Two Pointer Technique**: Use for finding middle, detecting cycles

Linked lists are fundamental data structures that provide flexible memory management and efficient insertion/deletion operations, making them essential for many algorithms and applications.
