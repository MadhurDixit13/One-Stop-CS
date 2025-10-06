# Pointers and Memory Management

## Overview
Pointers are variables that store memory addresses of other variables. They are fundamental to low-level programming and memory management, enabling dynamic memory allocation, data structures, and efficient algorithms. Understanding pointers is crucial for systems programming, embedded systems, and performance-critical applications.

## Basic Pointer Concepts

### What are Pointers?
```c
int x = 42;        // Variable x stores value 42
int *ptr = &x;     // Pointer ptr stores address of x
int value = *ptr;  // Dereference ptr to get value at address
```

### Memory Layout
```
Memory Address | Variable | Value
---------------|----------|-------
0x1000         | x        | 42
0x1004         | ptr      | 0x1000
0x1008         | value    | 42
```

## Pointer Operations

### 1. Declaration and Initialization
```c
// Basic pointer declaration
int *ptr;           // Pointer to integer
char *cptr;         // Pointer to character
float *fptr;        // Pointer to float

// Initialization
int x = 10;
int *ptr = &x;      // ptr points to x
int *ptr2 = NULL;   // Null pointer

// Pointer to pointer
int **pptr = &ptr;  // Pointer to pointer to int
```

### 2. Dereferencing
```c
int x = 42;
int *ptr = &x;

printf("Value of x: %d\n", x);           // 42
printf("Address of x: %p\n", &x);        // 0x1000
printf("Value via ptr: %d\n", *ptr);     // 42
printf("Address in ptr: %p\n", ptr);     // 0x1000

// Modifying value through pointer
*ptr = 100;
printf("New value of x: %d\n", x);       // 100
```

### 3. Pointer Arithmetic
```c
int arr[] = {10, 20, 30, 40, 50};
int *ptr = arr;  // Points to first element

printf("arr[0]: %d\n", *ptr);        // 10
printf("arr[1]: %d\n", *(ptr + 1));  // 20
printf("arr[2]: %d\n", *(ptr + 2));  // 30

// Increment pointer
ptr++;
printf("After increment: %d\n", *ptr); // 20

// Pointer difference
int *ptr2 = &arr[4];
printf("Elements between: %ld\n", ptr2 - ptr); // 4
```

## Dynamic Memory Management

### 1. malloc() - Memory Allocation
```c
#include <stdlib.h>

// Allocate memory for single integer
int *ptr = (int*)malloc(sizeof(int));
if (ptr == NULL) {
    printf("Memory allocation failed\n");
    exit(1);
}
*ptr = 42;

// Allocate memory for array
int n = 5;
int *arr = (int*)malloc(n * sizeof(int));
if (arr == NULL) {
    printf("Memory allocation failed\n");
    exit(1);
}

// Initialize array
for (int i = 0; i < n; i++) {
    arr[i] = i * 10;
}

// Free memory
free(ptr);
free(arr);
```

### 2. calloc() - Contiguous Allocation
```c
// Allocate and initialize memory to zero
int *arr = (int*)calloc(5, sizeof(int));
// Equivalent to: int *arr = (int*)malloc(5 * sizeof(int));
//                memset(arr, 0, 5 * sizeof(int));

// Check allocation
if (arr == NULL) {
    printf("Memory allocation failed\n");
    exit(1);
}

// Memory is initialized to zero
for (int i = 0; i < 5; i++) {
    printf("arr[%d]: %d\n", i, arr[i]); // All zeros
}

free(arr);
```

### 3. realloc() - Reallocation
```c
// Initial allocation
int *arr = (int*)malloc(3 * sizeof(int));
arr[0] = 1; arr[1] = 2; arr[2] = 3;

// Reallocate to larger size
int *new_arr = (int*)realloc(arr, 5 * sizeof(int));
if (new_arr == NULL) {
    printf("Reallocation failed\n");
    free(arr);
    exit(1);
}

arr = new_arr; // Update pointer
arr[3] = 4; arr[4] = 5;

// Shrink allocation
arr = (int*)realloc(arr, 2 * sizeof(int));

free(arr);
```

## Advanced Pointer Concepts

### 1. Pointer to Pointer
```c
int x = 42;
int *ptr = &x;
int **pptr = &ptr;

printf("x: %d\n", x);              // 42
printf("*ptr: %d\n", *ptr);        // 42
printf("**pptr: %d\n", **pptr);    // 42

// Modify through double pointer
**pptr = 100;
printf("Modified x: %d\n", x);     // 100
```

### 2. Function Pointers
```c
#include <stdio.h>

int add(int a, int b) { return a + b; }
int multiply(int a, int b) { return a * b; }

int main() {
    // Function pointer declaration
    int (*operation)(int, int);
    
    // Point to add function
    operation = add;
    printf("5 + 3 = %d\n", operation(5, 3)); // 8
    
    // Point to multiply function
    operation = multiply;
    printf("5 * 3 = %d\n", operation(5, 3)); // 15
    
    return 0;
}

// Array of function pointers
int (*operations[])(int, int) = {add, multiply};
```

### 3. Pointer Arrays
```c
int a = 1, b = 2, c = 3;
int *ptr_array[] = {&a, &b, &c};

for (int i = 0; i < 3; i++) {
    printf("ptr_array[%d] = %d\n", i, *ptr_array[i]);
}

// Array of strings (char pointers)
char *names[] = {"Alice", "Bob", "Charlie"};
for (int i = 0; i < 3; i++) {
    printf("Name %d: %s\n", i, names[i]);
}
```

## Common Pointer Pitfalls

### 1. Dangling Pointers
```c
int *ptr;

{
    int x = 42;
    ptr = &x;  // ptr points to local variable x
} // x goes out of scope here

// ptr is now a dangling pointer
// *ptr is undefined behavior
printf("%d\n", *ptr); // DON'T DO THIS!
```

### 2. Memory Leaks
```c
void function_with_leak() {
    int *ptr = malloc(sizeof(int));
    *ptr = 42;
    // Forgot to free(ptr) - MEMORY LEAK!
}

void correct_function() {
    int *ptr = malloc(sizeof(int));
    *ptr = 42;
    free(ptr);  // Always free allocated memory
}
```

### 3. Double Free
```c
int *ptr = malloc(sizeof(int));
free(ptr);
free(ptr);  // ERROR: Double free - undefined behavior!
```

### 4. Null Pointer Dereference
```c
int *ptr = NULL;
printf("%d\n", *ptr);  // ERROR: Dereferencing null pointer
```

### 5. Buffer Overflow
```c
char buffer[10];
char *ptr = buffer;

// Writing beyond buffer bounds
for (int i = 0; i < 20; i++) {
    ptr[i] = 'A';  // Buffer overflow!
}
```

## Safe Memory Management Patterns

### 1. RAII (Resource Acquisition Is Initialization)
```c
typedef struct {
    int *data;
    size_t size;
} SafeArray;

SafeArray* create_safe_array(size_t size) {
    SafeArray *arr = malloc(sizeof(SafeArray));
    if (!arr) return NULL;
    
    arr->data = malloc(size * sizeof(int));
    if (!arr->data) {
        free(arr);
        return NULL;
    }
    
    arr->size = size;
    return arr;
}

void destroy_safe_array(SafeArray *arr) {
    if (arr) {
        free(arr->data);
        free(arr);
    }
}

void use_safe_array() {
    SafeArray *arr = create_safe_array(10);
    if (!arr) {
        printf("Failed to create array\n");
        return;
    }
    
    // Use array...
    
    destroy_safe_array(arr);  // Always cleanup
}
```

### 2. Smart Pointer Pattern (C)
```c
typedef struct {
    int *data;
    int ref_count;
} SmartPointer;

SmartPointer* create_smart_pointer(int value) {
    SmartPointer *sp = malloc(sizeof(SmartPointer));
    if (!sp) return NULL;
    
    sp->data = malloc(sizeof(int));
    if (!sp->data) {
        free(sp);
        return NULL;
    }
    
    *(sp->data) = value;
    sp->ref_count = 1;
    return sp;
}

SmartPointer* copy_smart_pointer(SmartPointer *sp) {
    if (!sp) return NULL;
    
    sp->ref_count++;
    return sp;
}

void destroy_smart_pointer(SmartPointer *sp) {
    if (!sp) return;
    
    sp->ref_count--;
    if (sp->ref_count == 0) {
        free(sp->data);
        free(sp);
    }
}
```

### 3. Memory Pool
```c
typedef struct {
    void *memory;
    size_t block_size;
    size_t total_blocks;
    size_t free_blocks;
    size_t *free_list;
} MemoryPool;

MemoryPool* create_memory_pool(size_t block_size, size_t num_blocks) {
    MemoryPool *pool = malloc(sizeof(MemoryPool));
    if (!pool) return NULL;
    
    pool->memory = malloc(block_size * num_blocks);
    if (!pool->memory) {
        free(pool);
        return NULL;
    }
    
    pool->block_size = block_size;
    pool->total_blocks = num_blocks;
    pool->free_blocks = num_blocks;
    pool->free_list = malloc(num_blocks * sizeof(size_t));
    
    // Initialize free list
    for (size_t i = 0; i < num_blocks; i++) {
        pool->free_list[i] = i;
    }
    
    return pool;
}

void* allocate_from_pool(MemoryPool *pool) {
    if (pool->free_blocks == 0) return NULL;
    
    size_t block_index = pool->free_list[--pool->free_blocks];
    return (char*)pool->memory + block_index * pool->block_size;
}

void deallocate_to_pool(MemoryPool *pool, void *ptr) {
    if (!ptr) return;
    
    size_t block_index = ((char*)ptr - (char*)pool->memory) / pool->block_size;
    pool->free_list[pool->free_blocks++] = block_index;
}

void destroy_memory_pool(MemoryPool *pool) {
    if (pool) {
        free(pool->memory);
        free(pool->free_list);
        free(pool);
    }
}
```

## Pointer-Based Data Structures

### 1. Dynamic Array
```c
typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} DynamicArray;

DynamicArray* create_dynamic_array(size_t initial_capacity) {
    DynamicArray *arr = malloc(sizeof(DynamicArray));
    if (!arr) return NULL;
    
    arr->data = malloc(initial_capacity * sizeof(int));
    if (!arr->data) {
        free(arr);
        return NULL;
    }
    
    arr->size = 0;
    arr->capacity = initial_capacity;
    return arr;
}

int append(DynamicArray *arr, int value) {
    if (arr->size >= arr->capacity) {
        // Resize array
        size_t new_capacity = arr->capacity * 2;
        int *new_data = realloc(arr->data, new_capacity * sizeof(int));
        if (!new_data) return 0; // Failed to resize
        
        arr->data = new_data;
        arr->capacity = new_capacity;
    }
    
    arr->data[arr->size++] = value;
    return 1; // Success
}

void destroy_dynamic_array(DynamicArray *arr) {
    if (arr) {
        free(arr->data);
        free(arr);
    }
}
```

### 2. Linked List Node
```c
typedef struct Node {
    int data;
    struct Node *next;
} Node;

Node* create_node(int data) {
    Node *node = malloc(sizeof(Node));
    if (!node) return NULL;
    
    node->data = data;
    node->next = NULL;
    return node;
}

void insert_at_beginning(Node **head, int data) {
    Node *new_node = create_node(data);
    if (!new_node) return;
    
    new_node->next = *head;
    *head = new_node;
}

void delete_list(Node **head) {
    while (*head) {
        Node *temp = *head;
        *head = (*head)->next;
        free(temp);
    }
}
```

### 3. Binary Tree Node
```c
typedef struct TreeNode {
    int data;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;

TreeNode* create_tree_node(int data) {
    TreeNode *node = malloc(sizeof(TreeNode));
    if (!node) return NULL;
    
    node->data = data;
    node->left = node->right = NULL;
    return node;
}

TreeNode* insert_bst(TreeNode *root, int data) {
    if (!root) {
        return create_tree_node(data);
    }
    
    if (data < root->data) {
        root->left = insert_bst(root->left, data);
    } else if (data > root->data) {
        root->right = insert_bst(root->right, data);
    }
    
    return root;
}

void destroy_tree(TreeNode *root) {
    if (root) {
        destroy_tree(root->left);
        destroy_tree(root->right);
        free(root);
    }
}
```

## Memory Debugging Tools

### 1. Valgrind (Linux/macOS)
```bash
# Compile with debug symbols
gcc -g -o program program.c

# Run with Valgrind
valgrind --leak-check=full --show-leak-kinds=all ./program

# Output example:
# ==12345== HEAP SUMMARY:
# ==12345==     in use at exit: 0 bytes in 0 blocks
# ==12345==   total heap usage: 1 allocs, 1 frees, 1,024 bytes allocated
# ==12345== All heap blocks were freed -- no leaks are possible
```

### 2. AddressSanitizer (GCC/Clang)
```bash
# Compile with AddressSanitizer
gcc -fsanitize=address -g -o program program.c

# Run program
./program

# Detects:
# - Buffer overflows
# - Use after free
# - Double free
# - Memory leaks
```

### 3. Custom Memory Debugging
```c
#include <stdio.h>
#include <stdlib.h>

static size_t allocated_bytes = 0;
static size_t allocation_count = 0;

void* debug_malloc(size_t size) {
    void *ptr = malloc(size + sizeof(size_t));
    if (!ptr) return NULL;
    
    *(size_t*)ptr = size;
    allocated_bytes += size;
    allocation_count++;
    
    printf("Malloc: %zu bytes, total: %zu bytes, count: %zu\n", 
           size, allocated_bytes, allocation_count);
    
    return (char*)ptr + sizeof(size_t);
}

void debug_free(void *ptr) {
    if (!ptr) return;
    
    size_t *size_ptr = (size_t*)((char*)ptr - sizeof(size_t));
    size_t size = *size_ptr;
    
    allocated_bytes -= size;
    allocation_count--;
    
    printf("Free: %zu bytes, remaining: %zu bytes, count: %zu\n", 
           size, allocated_bytes, allocation_count);
    
    free(size_ptr);
}

#define malloc(size) debug_malloc(size)
#define free(ptr) debug_free(ptr)
```

## Best Practices

### 1. Always Initialize Pointers
```c
// Good
int *ptr = NULL;

// Bad
int *ptr;  // Uninitialized pointer
```

### 2. Check for NULL Before Use
```c
if (ptr != NULL) {
    *ptr = 42;
}
```

### 3. Free Memory in Reverse Order
```c
// Allocate
char *str1 = malloc(100);
char *str2 = malloc(200);

// Free in reverse order
free(str2);
free(str1);
```

### 4. Use const Correctness
```c
// Pointer to const data
const int *ptr1 = &x;
// *ptr1 = 10;  // Error: cannot modify const data

// Const pointer to data
int *const ptr2 = &x;
ptr2 = &y;  // Error: cannot change pointer
*ptr2 = 10; // OK: can modify data

// Const pointer to const data
const int *const ptr3 = &x;
// ptr3 = &y;  // Error
// *ptr3 = 10; // Error
```

### 5. Avoid Pointer Arithmetic on void*
```c
// Bad
void *ptr = malloc(100);
ptr++;  // Error: arithmetic on void*

// Good
char *ptr = malloc(100);
ptr++;  // OK: arithmetic on char*
```

Pointers are powerful tools that require careful management. Understanding memory layout, proper allocation/deallocation, and common pitfalls is essential for writing robust, efficient C programs.
