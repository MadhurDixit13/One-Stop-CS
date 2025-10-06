# Trees Data Structure

## Overview
A tree is a hierarchical data structure consisting of nodes connected by edges. Each tree has a root node and nodes can have children, forming a parent-child relationship. Trees are fundamental data structures used in many algorithms and applications.

## Tree Terminology

### Basic Terms
- **Node**: Each element in the tree
- **Root**: The topmost node (has no parent)
- **Leaf**: Node with no children
- **Parent**: Node that has children
- **Child**: Node directly connected to a parent
- **Sibling**: Nodes with the same parent
- **Ancestor**: All nodes from root to current node's parent
- **Descendant**: All nodes in subtree rooted at current node
- **Depth**: Distance from root to node
- **Height**: Distance from node to deepest leaf
- **Degree**: Number of children a node has

### Tree Structure Example
```
        A (root, height=3, depth=0)
       / \
      B   C (siblings, height=2, depth=1)
     / \   \
    D   E   F (leaves, height=0, depth=2)
```

## Types of Trees

### 1. Binary Tree
Each node has at most two children (left and right).

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class BinaryTree:
    def __init__(self):
        self.root = None
    
    def insert(self, val):
        """Insert value into binary tree."""
        if not self.root:
            self.root = TreeNode(val)
            return
        
        queue = [self.root]
        while queue:
            node = queue.pop(0)
            
            if not node.left:
                node.left = TreeNode(val)
                return
            if not node.right:
                node.right = TreeNode(val)
                return
            
            queue.append(node.left)
            queue.append(node.right)
    
    def inorder_traversal(self, root):
        """Inorder: Left -> Root -> Right"""
        result = []
        if root:
            result.extend(self.inorder_traversal(root.left))
            result.append(root.val)
            result.extend(self.inorder_traversal(root.right))
        return result
    
    def preorder_traversal(self, root):
        """Preorder: Root -> Left -> Right"""
        result = []
        if root:
            result.append(root.val)
            result.extend(self.preorder_traversal(root.left))
            result.extend(self.preorder_traversal(root.right))
        return result
    
    def postorder_traversal(self, root):
        """Postorder: Left -> Right -> Root"""
        result = []
        if root:
            result.extend(self.postorder_traversal(root.left))
            result.extend(self.postorder_traversal(root.right))
            result.append(root.val)
        return result
    
    def level_order_traversal(self, root):
        """Level order (BFS): Visit nodes level by level"""
        if not root:
            return []
        
        result = []
        queue = [root]
        
        while queue:
            level_size = len(queue)
            level = []
            
            for _ in range(level_size):
                node = queue.pop(0)
                level.append(node.val)
                
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)
            
            result.append(level)
        
        return result
```

### 2. Binary Search Tree (BST)
Binary tree where left child < parent < right child.

```python
class BinarySearchTree:
    def __init__(self):
        self.root = None
    
    def insert(self, val):
        """Insert value into BST."""
        self.root = self._insert_recursive(self.root, val)
    
    def _insert_recursive(self, root, val):
        if not root:
            return TreeNode(val)
        
        if val < root.val:
            root.left = self._insert_recursive(root.left, val)
        else:
            root.right = self._insert_recursive(root.right, val)
        
        return root
    
    def search(self, val):
        """Search for value in BST."""
        return self._search_recursive(self.root, val)
    
    def _search_recursive(self, root, val):
        if not root or root.val == val:
            return root
        
        if val < root.val:
            return self._search_recursive(root.left, val)
        else:
            return self._search_recursive(root.right, val)
    
    def delete(self, val):
        """Delete value from BST."""
        self.root = self._delete_recursive(self.root, val)
    
    def _delete_recursive(self, root, val):
        if not root:
            return root
        
        if val < root.val:
            root.left = self._delete_recursive(root.left, val)
        elif val > root.val:
            root.right = self._delete_recursive(root.right, val)
        else:
            # Node to be deleted found
            if not root.left:
                return root.right
            elif not root.right:
                return root.left
            
            # Node with two children
            # Get inorder successor (smallest in right subtree)
            root.val = self._min_value(root.right)
            
            # Delete the inorder successor
            root.right = self._delete_recursive(root.right, root.val)
        
        return root
    
    def _min_value(self, root):
        """Find minimum value in BST."""
        while root.left:
            root = root.left
        return root.val
    
    def is_valid_bst(self, root):
        """Check if tree is valid BST."""
        def validate(node, min_val, max_val):
            if not node:
                return True
            
            if node.val <= min_val or node.val >= max_val:
                return False
            
            return (validate(node.left, min_val, node.val) and
                    validate(node.right, node.val, max_val))
        
        return validate(root, float('-inf'), float('inf'))
```

### 3. AVL Tree (Self-Balancing BST)
Maintains balance by ensuring height difference between subtrees ≤ 1.

```python
class AVLNode:
    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None
        self.height = 1

class AVLTree:
    def __init__(self):
        self.root = None
    
    def get_height(self, node):
        if not node:
            return 0
        return node.height
    
    def get_balance(self, node):
        if not node:
            return 0
        return self.get_height(node.left) - self.get_height(node.right)
    
    def update_height(self, node):
        node.height = 1 + max(self.get_height(node.left), self.get_height(node.right))
    
    def rotate_right(self, y):
        """Right rotation for AVL tree."""
        x = y.left
        T2 = x.right
        
        # Perform rotation
        x.right = y
        y.left = T2
        
        # Update heights
        self.update_height(y)
        self.update_height(x)
        
        return x
    
    def rotate_left(self, x):
        """Left rotation for AVL tree."""
        y = x.right
        T2 = y.left
        
        # Perform rotation
        y.left = x
        x.right = T2
        
        # Update heights
        self.update_height(x)
        self.update_height(y)
        
        return y
    
    def insert(self, val):
        """Insert value into AVL tree."""
        self.root = self._insert_recursive(self.root, val)
    
    def _insert_recursive(self, node, val):
        # 1. Perform normal BST insertion
        if not node:
            return AVLNode(val)
        
        if val < node.val:
            node.left = self._insert_recursive(node.left, val)
        else:
            node.right = self._insert_recursive(node.right, val)
        
        # 2. Update height of ancestor node
        self.update_height(node)
        
        # 3. Get balance factor
        balance = self.get_balance(node)
        
        # 4. If unbalanced, perform rotations
        # Left Left Case
        if balance > 1 and val < node.left.val:
            return self.rotate_right(node)
        
        # Right Right Case
        if balance < -1 and val > node.right.val:
            return self.rotate_left(node)
        
        # Left Right Case
        if balance > 1 and val > node.left.val:
            node.left = self.rotate_left(node.left)
            return self.rotate_right(node)
        
        # Right Left Case
        if balance < -1 and val < node.right.val:
            node.right = self.rotate_right(node.right)
            return self.rotate_left(node)
        
        return node
```

### 4. Red-Black Tree
Self-balancing BST with color properties for balance.

```python
class RedBlackNode:
    def __init__(self, val, color='RED'):
        self.val = val
        self.left = None
        self.right = None
        self.parent = None
        self.color = color  # 'RED' or 'BLACK'

class RedBlackTree:
    def __init__(self):
        self.NIL = RedBlackNode(0, 'BLACK')  # Sentinel node
        self.root = self.NIL
    
    def insert(self, val):
        """Insert value into Red-Black tree."""
        new_node = RedBlackNode(val)
        new_node.left = self.NIL
        new_node.right = self.NIL
        
        # Find insertion point
        parent = None
        current = self.root
        
        while current != self.NIL:
            parent = current
            if val < current.val:
                current = current.left
            else:
                current = current.right
        
        new_node.parent = parent
        
        if parent is None:
            self.root = new_node
        elif val < parent.val:
            parent.left = new_node
        else:
            parent.right = new_node
        
        # Fix Red-Black properties
        self._fix_insert(new_node)
    
    def _fix_insert(self, node):
        """Fix Red-Black tree properties after insertion."""
        while node.parent and node.parent.color == 'RED':
            if node.parent == node.parent.parent.left:
                uncle = node.parent.parent.right
                
                if uncle.color == 'RED':
                    # Case 1: Uncle is red
                    node.parent.color = 'BLACK'
                    uncle.color = 'BLACK'
                    node.parent.parent.color = 'RED'
                    node = node.parent.parent
                else:
                    if node == node.parent.right:
                        # Case 2: Uncle is black, node is right child
                        node = node.parent
                        self._rotate_left(node)
                    
                    # Case 3: Uncle is black, node is left child
                    node.parent.color = 'BLACK'
                    node.parent.parent.color = 'RED'
                    self._rotate_right(node.parent.parent)
            else:
                # Mirror cases
                uncle = node.parent.parent.left
                
                if uncle.color == 'RED':
                    node.parent.color = 'BLACK'
                    uncle.color = 'BLACK'
                    node.parent.parent.color = 'RED'
                    node = node.parent.parent
                else:
                    if node == node.parent.left:
                        node = node.parent
                        self._rotate_right(node)
                    
                    node.parent.color = 'BLACK'
                    node.parent.parent.color = 'RED'
                    self._rotate_left(node.parent.parent)
        
        self.root.color = 'BLACK'
    
    def _rotate_left(self, x):
        """Left rotation."""
        y = x.right
        x.right = y.left
        
        if y.left != self.NIL:
            y.left.parent = x
        
        y.parent = x.parent
        
        if x.parent is None:
            self.root = y
        elif x == x.parent.left:
            x.parent.left = y
        else:
            x.parent.right = y
        
        y.left = x
        x.parent = y
    
    def _rotate_right(self, y):
        """Right rotation."""
        x = y.left
        y.left = x.right
        
        if x.right != self.NIL:
            x.right.parent = y
        
        x.parent = y.parent
        
        if y.parent is None:
            self.root = x
        elif y == y.parent.right:
            y.parent.right = x
        else:
            y.parent.left = x
        
        x.right = y
        y.parent = x
```

### 5. B-Tree
Multi-way tree for disk storage optimization.

```python
class BTreeNode:
    def __init__(self, is_leaf=False):
        self.keys = []
        self.children = []
        self.is_leaf = is_leaf

class BTree:
    def __init__(self, t):
        self.root = BTreeNode(True)
        self.t = t  # Minimum degree
    
    def search(self, k):
        """Search for key k in B-tree."""
        return self._search_recursive(self.root, k)
    
    def _search_recursive(self, node, k):
        i = 0
        while i < len(node.keys) and k > node.keys[i]:
            i += 1
        
        if i < len(node.keys) and node.keys[i] == k:
            return node
        
        if node.is_leaf:
            return None
        
        return self._search_recursive(node.children[i], k)
    
    def insert(self, k):
        """Insert key k into B-tree."""
        root = self.root
        
        if len(root.keys) == 2 * self.t - 1:
            # Root is full, split it
            new_root = BTreeNode(False)
            new_root.children.append(root)
            self._split_child(new_root, 0)
            self.root = new_root
            self._insert_non_full(new_root, k)
        else:
            self._insert_non_full(root, k)
    
    def _insert_non_full(self, node, k):
        """Insert key into non-full node."""
        i = len(node.keys) - 1
        
        if node.is_leaf:
            node.keys.append(0)
            while i >= 0 and node.keys[i] > k:
                node.keys[i + 1] = node.keys[i]
                i -= 1
            node.keys[i + 1] = k
        else:
            while i >= 0 and node.keys[i] > k:
                i -= 1
            i += 1
            
            if len(node.children[i].keys) == 2 * self.t - 1:
                self._split_child(node, i)
                if node.keys[i] < k:
                    i += 1
            
            self._insert_non_full(node.children[i], k)
    
    def _split_child(self, parent, index):
        """Split full child node."""
        t = self.t
        y = parent.children[index]
        z = BTreeNode(y.is_leaf)
        
        # Move keys
        z.keys = y.keys[t:]
        y.keys = y.keys[:t-1]
        
        # Move children if not leaf
        if not y.is_leaf:
            z.children = y.children[t:]
            y.children = y.children[:t]
        
        # Insert z into parent
        parent.children.insert(index + 1, z)
        parent.keys.insert(index, y.keys[t-1])
        
        y.keys = y.keys[:t-1]
```

## Tree Algorithms

### Tree Traversal Algorithms

```python
def iterative_inorder(root):
    """Iterative inorder traversal."""
    result = []
    stack = []
    current = root
    
    while stack or current:
        while current:
            stack.append(current)
            current = current.left
        
        current = stack.pop()
        result.append(current.val)
        current = current.right
    
    return result

def iterative_preorder(root):
    """Iterative preorder traversal."""
    if not root:
        return []
    
    result = []
    stack = [root]
    
    while stack:
        node = stack.pop()
        result.append(node.val)
        
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)
    
    return result

def iterative_postorder(root):
    """Iterative postorder traversal."""
    if not root:
        return []
    
    result = []
    stack = [root]
    
    while stack:
        node = stack.pop()
        result.append(node.val)
        
        if node.left:
            stack.append(node.left)
        if node.right:
            stack.append(node.right)
    
    return result[::-1]  # Reverse for postorder
```

### Tree Properties and Validation

```python
def max_depth(root):
    """Calculate maximum depth of binary tree."""
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))

def is_balanced(root):
    """Check if binary tree is height-balanced."""
    def check_height(node):
        if not node:
            return 0
        
        left_height = check_height(node.left)
        if left_height == -1:
            return -1
        
        right_height = check_height(node.right)
        if right_height == -1:
            return -1
        
        if abs(left_height - right_height) > 1:
            return -1
        
        return 1 + max(left_height, right_height)
    
    return check_height(root) != -1

def is_symmetric(root):
    """Check if binary tree is symmetric."""
    def is_mirror(left, right):
        if not left and not right:
            return True
        if not left or not right:
            return False
        return (left.val == right.val and
                is_mirror(left.left, right.right) and
                is_mirror(left.right, right.left))
    
    if not root:
        return True
    return is_mirror(root.left, root.right)

def count_nodes(root):
    """Count total nodes in binary tree."""
    if not root:
        return 0
    return 1 + count_nodes(root.left) + count_nodes(root.right)

def sum_of_path_numbers(root):
    """Calculate sum of all path numbers from root to leaf."""
    def dfs(node, path_sum):
        if not node:
            return 0
        
        path_sum = path_sum * 10 + node.val
        
        if not node.left and not node.right:
            return path_sum
        
        return dfs(node.left, path_sum) + dfs(node.right, path_sum)
    
    return dfs(root, 0)
```

### Path and Subtree Problems

```python
def has_path_sum(root, target_sum):
    """Check if tree has root-to-leaf path with given sum."""
    if not root:
        return False
    
    if not root.left and not root.right:
        return root.val == target_sum
    
    return (has_path_sum(root.left, target_sum - root.val) or
            has_path_sum(root.right, target_sum - root.val))

def path_sum_paths(root, target_sum):
    """Find all root-to-leaf paths with given sum."""
    def dfs(node, target, path, result):
        if not node:
            return
        
        path.append(node.val)
        
        if not node.left and not node.right and node.val == target:
            result.append(path[:])
        else:
            dfs(node.left, target - node.val, path, result)
            dfs(node.right, target - node.val, path, result)
        
        path.pop()
    
    result = []
    dfs(root, target_sum, [], result)
    return result

def lowest_common_ancestor(root, p, q):
    """Find lowest common ancestor of two nodes."""
    if not root or root == p or root == q:
        return root
    
    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)
    
    if left and right:
        return root
    
    return left or right

def diameter_of_binary_tree(root):
    """Calculate diameter of binary tree."""
    def dfs(node):
        if not node:
            return 0
        
        left_depth = dfs(node.left)
        right_depth = dfs(node.right)
        
        # Update diameter
        diameter[0] = max(diameter[0], left_depth + right_depth)
        
        return 1 + max(left_depth, right_depth)
    
    diameter = [0]
    dfs(root)
    return diameter[0]
```

## Performance Analysis

### Time Complexity Comparison

| Operation | BST | AVL Tree | Red-Black Tree | B-Tree |
|-----------|-----|----------|----------------|--------|
| Search | O(h) | O(log n) | O(log n) | O(log n) |
| Insert | O(h) | O(log n) | O(log n) | O(log n) |
| Delete | O(h) | O(log n) | O(log n) | O(log n) |
| Space | O(n) | O(n) | O(n) | O(n) |

Where h is height of tree (h = log n for balanced trees)

## Applications

### Real-world Uses
1. **File Systems**: Directory structures, B-trees for file indexing
2. **Databases**: Index structures, query optimization
3. **Compilers**: Parse trees, syntax analysis
4. **Networking**: Routing tables, decision trees
5. **AI/ML**: Decision trees, neural network structures
6. **Operating Systems**: Process trees, memory management
7. **Graphics**: Scene graphs, spatial data structures

### When to Use Different Trees
- **Binary Tree**: Simple hierarchical data
- **BST**: Sorted data with fast search
- **AVL Tree**: When frequent lookups and balanced height needed
- **Red-Black Tree**: When insertions/deletions are frequent
- **B-Tree**: Disk-based storage, databases

Trees are fundamental data structures that provide efficient ways to organize and search hierarchical data, forming the basis for many advanced algorithms and systems.
