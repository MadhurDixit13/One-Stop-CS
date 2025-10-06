# Graphs Data Structure

## Overview
A graph is a non-linear data structure consisting of vertices (nodes) and edges that connect these vertices. Graphs are used to represent relationships between objects and are fundamental in computer science for modeling networks, social connections, maps, and more.

## Graph Terminology

### Basic Terms
- **Vertex (Node)**: A fundamental unit of a graph
- **Edge**: Connection between two vertices
- **Adjacent**: Two vertices connected by an edge
- **Degree**: Number of edges incident to a vertex
- **Path**: Sequence of vertices connected by edges
- **Cycle**: Path that starts and ends at the same vertex
- **Connected**: Path exists between every pair of vertices
- **Weight**: Value associated with an edge (weighted graphs)

### Graph Types
```
Undirected Graph: A-B, B-C, C-A
Directed Graph: A→B, B→C, C→A
Weighted Graph: A--5--B, B--3--C, C--2--A
```

## Graph Representation

### 1. Adjacency Matrix
```python
class GraphAdjacencyMatrix:
    def __init__(self, vertices):
        self.vertices = vertices
        self.matrix = [[0] * vertices for _ in range(vertices)]
    
    def add_edge(self, u, v, weight=1):
        """Add edge between vertices u and v."""
        self.matrix[u][v] = weight
        self.matrix[v][u] = weight  # For undirected graph
    
    def remove_edge(self, u, v):
        """Remove edge between vertices u and v."""
        self.matrix[u][v] = 0
        self.matrix[v][u] = 0
    
    def has_edge(self, u, v):
        """Check if edge exists between u and v."""
        return self.matrix[u][v] != 0
    
    def get_neighbors(self, vertex):
        """Get all neighbors of a vertex."""
        neighbors = []
        for i in range(self.vertices):
            if self.matrix[vertex][i] != 0:
                neighbors.append(i)
        return neighbors
    
    def print_graph(self):
        """Print adjacency matrix."""
        for row in self.matrix:
            print(row)

# Example usage
graph = GraphAdjacencyMatrix(4)
graph.add_edge(0, 1, 5)
graph.add_edge(0, 2, 3)
graph.add_edge(1, 3, 2)
graph.print_graph()
```

### 2. Adjacency List
```python
from collections import defaultdict

class GraphAdjacencyList:
    def __init__(self):
        self.graph = defaultdict(list)
        self.vertices = set()
    
    def add_edge(self, u, v, weight=1):
        """Add edge between vertices u and v."""
        self.graph[u].append((v, weight))
        self.graph[v].append((u, weight))  # For undirected graph
        self.vertices.add(u)
        self.vertices.add(v)
    
    def remove_edge(self, u, v):
        """Remove edge between vertices u and v."""
        self.graph[u] = [(neighbor, weight) for neighbor, weight in self.graph[u] if neighbor != v]
        self.graph[v] = [(neighbor, weight) for neighbor, weight in self.graph[v] if neighbor != u]
    
    def get_neighbors(self, vertex):
        """Get all neighbors of a vertex."""
        return self.graph[vertex]
    
    def print_graph(self):
        """Print adjacency list."""
        for vertex in sorted(self.vertices):
            neighbors = [f"{neighbor}({weight})" for neighbor, weight in self.graph[vertex]]
            print(f"{vertex}: {' '.join(neighbors)}")
    
    def get_vertices(self):
        """Get all vertices."""
        return list(self.vertices)
    
    def get_edges(self):
        """Get all edges."""
        edges = set()
        for vertex in self.vertices:
            for neighbor, weight in self.graph[vertex]:
                edge = tuple(sorted([vertex, neighbor])) if vertex < neighbor else tuple(sorted([neighbor, vertex]))
                edges.add((edge[0], edge[1], weight))
        return list(edges)

# Example usage
graph = GraphAdjacencyList()
graph.add_edge(0, 1, 5)
graph.add_edge(0, 2, 3)
graph.add_edge(1, 3, 2)
graph.print_graph()
```

### 3. Edge List
```python
class GraphEdgeList:
    def __init__(self):
        self.edges = []
        self.vertices = set()
    
    def add_edge(self, u, v, weight=1):
        """Add edge between vertices u and v."""
        self.edges.append((u, v, weight))
        self.vertices.add(u)
        self.vertices.add(v)
    
    def remove_edge(self, u, v):
        """Remove edge between vertices u and v."""
        self.edges = [(src, dest, weight) for src, dest, weight in self.edges 
                     if not (src == u and dest == v) and not (src == v and dest == u)]
    
    def get_neighbors(self, vertex):
        """Get all neighbors of a vertex."""
        neighbors = []
        for src, dest, weight in self.edges:
            if src == vertex:
                neighbors.append((dest, weight))
            elif dest == vertex:
                neighbors.append((src, weight))
        return neighbors
    
    def print_graph(self):
        """Print edge list."""
        for u, v, weight in self.edges:
            print(f"{u} --{weight}-- {v}")
```

## Graph Traversal Algorithms

### 1. Depth-First Search (DFS)
```python
def dfs_recursive(graph, start, visited=None):
    """DFS using recursion."""
    if visited is None:
        visited = set()
    
    visited.add(start)
    print(start, end=" ")
    
    for neighbor, _ in graph.get_neighbors(start):
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited)
    
    return visited

def dfs_iterative(graph, start):
    """DFS using iteration and stack."""
    visited = set()
    stack = [start]
    
    while stack:
        vertex = stack.pop()
        
        if vertex not in visited:
            visited.add(vertex)
            print(vertex, end=" ")
            
            # Add neighbors in reverse order for consistent traversal
            neighbors = [neighbor for neighbor, _ in graph.get_neighbors(vertex)]
            for neighbor in reversed(neighbors):
                if neighbor not in visited:
                    stack.append(neighbor)
    
    return visited

def dfs_path(graph, start, end):
    """Find path from start to end using DFS."""
    def dfs_helper(current, target, path, visited):
        if current == target:
            return path + [current]
        
        visited.add(current)
        
        for neighbor, _ in graph.get_neighbors(current):
            if neighbor not in visited:
                result = dfs_helper(neighbor, target, path + [current], visited)
                if result:
                    return result
        
        return None
    
    return dfs_helper(start, end, [], set())
```

### 2. Breadth-First Search (BFS)
```python
from collections import deque

def bfs(graph, start):
    """BFS using queue."""
    visited = set()
    queue = deque([start])
    visited.add(start)
    
    while queue:
        vertex = queue.popleft()
        print(vertex, end=" ")
        
        for neighbor, _ in graph.get_neighbors(vertex):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    
    return visited

def bfs_path(graph, start, end):
    """Find shortest path using BFS."""
    if start == end:
        return [start]
    
    visited = set()
    queue = deque([(start, [start])])
    visited.add(start)
    
    while queue:
        vertex, path = queue.popleft()
        
        for neighbor, _ in graph.get_neighbors(vertex):
            if neighbor == end:
                return path + [neighbor]
            
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))
    
    return None  # No path found

def bfs_level_order(graph, start):
    """BFS with level-by-level traversal."""
    if start not in graph.vertices:
        return []
    
    visited = set()
    queue = deque([start])
    visited.add(start)
    levels = [[start]]
    
    while queue:
        level_size = len(queue)
        current_level = []
        
        for _ in range(level_size):
            vertex = queue.popleft()
            
            for neighbor, _ in graph.get_neighbors(vertex):
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.append(neighbor)
                    current_level.append(neighbor)
        
        if current_level:
            levels.append(current_level)
    
    return levels
```

## Shortest Path Algorithms

### 1. Dijkstra's Algorithm
```python
import heapq

def dijkstra(graph, start):
    """Find shortest distances from start to all vertices."""
    distances = {vertex: float('inf') for vertex in graph.get_vertices()}
    distances[start] = 0
    previous = {}
    
    # Priority queue: (distance, vertex)
    pq = [(0, start)]
    visited = set()
    
    while pq:
        current_dist, current_vertex = heapq.heappop(pq)
        
        if current_vertex in visited:
            continue
        
        visited.add(current_vertex)
        
        for neighbor, weight in graph.get_neighbors(current_vertex):
            if neighbor in visited:
                continue
            
            distance = current_dist + weight
            
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                previous[neighbor] = current_vertex
                heapq.heappush(pq, (distance, neighbor))
    
    return distances, previous

def dijkstra_path(graph, start, end):
    """Find shortest path from start to end."""
    distances, previous = dijkstra(graph, start)
    
    if distances[end] == float('inf'):
        return None, float('inf')
    
    # Reconstruct path
    path = []
    current = end
    
    while current is not None:
        path.append(current)
        current = previous.get(current)
    
    return path[::-1], distances[end]
```

### 2. Bellman-Ford Algorithm
```python
def bellman_ford(graph, start):
    """Find shortest distances using Bellman-Ford algorithm."""
    vertices = graph.get_vertices()
    distances = {vertex: float('inf') for vertex in vertices}
    distances[start] = 0
    previous = {}
    
    # Relax edges V-1 times
    for _ in range(len(vertices) - 1):
        for u, v, weight in graph.get_edges():
            if distances[u] != float('inf') and distances[u] + weight < distances[v]:
                distances[v] = distances[u] + weight
                previous[v] = u
    
    # Check for negative cycles
    for u, v, weight in graph.get_edges():
        if distances[u] != float('inf') and distances[u] + weight < distances[v]:
            raise ValueError("Graph contains negative cycle")
    
    return distances, previous
```

### 3. Floyd-Warshall Algorithm
```python
def floyd_warshall(graph):
    """Find shortest distances between all pairs of vertices."""
    vertices = graph.get_vertices()
    n = len(vertices)
    
    # Initialize distance matrix
    dist = [[float('inf')] * n for _ in range(n)]
    next_vertex = [[None] * n for _ in range(n)]
    
    # Distance from vertex to itself is 0
    for i in range(n):
        dist[i][i] = 0
    
    # Add edge weights
    for u, v, weight in graph.get_edges():
        u_idx = vertices.index(u)
        v_idx = vertices.index(v)
        dist[u_idx][v_idx] = weight
        next_vertex[u_idx][v_idx] = v_idx
    
    # Floyd-Warshall algorithm
    for k in range(n):
        for i in range(n):
            for j in range(n):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]
                    next_vertex[i][j] = next_vertex[i][k]
    
    return dist, next_vertex, vertices
```

## Minimum Spanning Tree Algorithms

### 1. Kruskal's Algorithm
```python
class UnionFind:
    def __init__(self, vertices):
        self.parent = {v: v for v in vertices}
        self.rank = {v: 0 for v in vertices}
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        
        if px == py:
            return False
        
        if self.rank[px] < self.rank[py]:
            self.parent[px] = py
        elif self.rank[px] > self.rank[py]:
            self.parent[py] = px
        else:
            self.parent[py] = px
            self.rank[px] += 1
        
        return True

def kruskal(graph):
    """Find minimum spanning tree using Kruskal's algorithm."""
    vertices = graph.get_vertices()
    edges = graph.get_edges()
    
    # Sort edges by weight
    edges.sort(key=lambda x: x[2])
    
    mst = []
    uf = UnionFind(vertices)
    
    for u, v, weight in edges:
        if uf.union(u, v):
            mst.append((u, v, weight))
            if len(mst) == len(vertices) - 1:
                break
    
    return mst
```

### 2. Prim's Algorithm
```python
def prim(graph, start):
    """Find minimum spanning tree using Prim's algorithm."""
    vertices = graph.get_vertices()
    mst = []
    visited = {start}
    
    # Priority queue: (weight, u, v)
    pq = []
    
    # Add edges from start vertex
    for neighbor, weight in graph.get_neighbors(start):
        heapq.heappush(pq, (weight, start, neighbor))
    
    while pq and len(visited) < len(vertices):
        weight, u, v = heapq.heappop(pq)
        
        if v in visited:
            continue
        
        visited.add(v)
        mst.append((u, v, weight))
        
        # Add edges from new vertex
        for neighbor, edge_weight in graph.get_neighbors(v):
            if neighbor not in visited:
                heapq.heappush(pq, (edge_weight, v, neighbor))
    
    return mst
```

## Topological Sorting

```python
def topological_sort_dfs(graph):
    """Topological sort using DFS."""
    visited = set()
    stack = []
    
    def dfs(vertex):
        visited.add(vertex)
        for neighbor, _ in graph.get_neighbors(vertex):
            if neighbor not in visited:
                dfs(neighbor)
        stack.append(vertex)
    
    for vertex in graph.get_vertices():
        if vertex not in visited:
            dfs(vertex)
    
    return stack[::-1]

def topological_sort_kahn(graph):
    """Topological sort using Kahn's algorithm (BFS)."""
    # Calculate in-degrees
    in_degree = {vertex: 0 for vertex in graph.get_vertices()}
    
    for vertex in graph.get_vertices():
        for neighbor, _ in graph.get_neighbors(vertex):
            in_degree[neighbor] += 1
    
    # Queue for vertices with no incoming edges
    queue = deque([v for v, degree in in_degree.items() if degree == 0])
    result = []
    
    while queue:
        vertex = queue.popleft()
        result.append(vertex)
        
        for neighbor, _ in graph.get_neighbors(vertex):
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)
    
    # Check for cycle
    if len(result) != len(graph.get_vertices()):
        raise ValueError("Graph contains cycle")
    
    return result
```

## Cycle Detection

```python
def has_cycle_undirected(graph):
    """Detect cycle in undirected graph using DFS."""
    visited = set()
    
    def dfs(vertex, parent):
        visited.add(vertex)
        
        for neighbor, _ in graph.get_neighbors(vertex):
            if neighbor not in visited:
                if dfs(neighbor, vertex):
                    return True
            elif neighbor != parent:
                return True
        
        return False
    
    for vertex in graph.get_vertices():
        if vertex not in visited:
            if dfs(vertex, None):
                return True
    
    return False

def has_cycle_directed(graph):
    """Detect cycle in directed graph using DFS."""
    WHITE, GRAY, BLACK = 0, 1, 2
    color = {vertex: WHITE for vertex in graph.get_vertices()}
    
    def dfs(vertex):
        color[vertex] = GRAY
        
        for neighbor, _ in graph.get_neighbors(vertex):
            if color[neighbor] == GRAY:
                return True
            if color[neighbor] == WHITE and dfs(neighbor):
                return True
        
        color[vertex] = BLACK
        return False
    
    for vertex in graph.get_vertices():
        if color[vertex] == WHITE:
            if dfs(vertex):
                return True
    
    return False
```

## Graph Coloring

```python
def graph_coloring_greedy(graph):
    """Color graph using greedy algorithm."""
    colors = {}
    
    for vertex in graph.get_vertices():
        used_colors = set()
        
        for neighbor, _ in graph.get_neighbors(vertex):
            if neighbor in colors:
                used_colors.add(colors[neighbor])
        
        # Find smallest available color
        color = 0
        while color in used_colors:
            color += 1
        
        colors[vertex] = color
    
    return colors

def is_bipartite(graph):
    """Check if graph is bipartite using BFS."""
    color = {}
    
    for vertex in graph.get_vertices():
        if vertex not in color:
            queue = deque([vertex])
            color[vertex] = 0
            
            while queue:
                current = queue.popleft()
                
                for neighbor, _ in graph.get_neighbors(current):
                    if neighbor not in color:
                        color[neighbor] = 1 - color[current]
                        queue.append(neighbor)
                    elif color[neighbor] == color[current]:
                        return False
    
    return True
```

## Strongly Connected Components

```python
def kosaraju_scc(graph):
    """Find strongly connected components using Kosaraju's algorithm."""
    visited = set()
    stack = []
    
    # First DFS to fill stack
    def dfs1(vertex):
        visited.add(vertex)
        for neighbor, _ in graph.get_neighbors(vertex):
            if neighbor not in visited:
                dfs1(neighbor)
        stack.append(vertex)
    
    for vertex in graph.get_vertices():
        if vertex not in visited:
            dfs1(vertex)
    
    # Create transpose graph
    transpose = GraphAdjacencyList()
    for u, v, weight in graph.get_edges():
        transpose.add_edge(v, u, weight)
    
    # Second DFS on transpose graph
    visited.clear()
    sccs = []
    
    def dfs2(vertex, component):
        visited.add(vertex)
        component.append(vertex)
        for neighbor, _ in transpose.get_neighbors(vertex):
            if neighbor not in visited:
                dfs2(neighbor, component)
    
    while stack:
        vertex = stack.pop()
        if vertex not in visited:
            component = []
            dfs2(vertex, component)
            sccs.append(component)
    
    return sccs
```

## Performance Analysis

### Time Complexity

| Algorithm | Time Complexity | Space Complexity |
|-----------|----------------|------------------|
| DFS/BFS | O(V + E) | O(V) |
| Dijkstra | O((V + E) log V) | O(V) |
| Bellman-Ford | O(VE) | O(V) |
| Floyd-Warshall | O(V³) | O(V²) |
| Kruskal | O(E log E) | O(V) |
| Prim | O((V + E) log V) | O(V) |
| Topological Sort | O(V + E) | O(V) |

Where V = vertices, E = edges

## Applications

### Real-world Uses
1. **Social Networks**: Friend connections, influence propagation
2. **Transportation**: Road networks, flight routes
3. **Computer Networks**: Internet topology, routing
4. **Game Development**: Pathfinding, AI decision trees
5. **Dependency Management**: Package dependencies, build systems
6. **Recommendation Systems**: User-item relationships
7. **Biology**: Protein interactions, gene networks

### When to Use Different Representations
- **Adjacency Matrix**: Dense graphs, frequent edge queries
- **Adjacency List**: Sparse graphs, memory efficiency
- **Edge List**: When you need to iterate over all edges

### Graph Types for Different Problems
- **Unweighted**: BFS, DFS, connectivity
- **Weighted**: Shortest path, MST algorithms
- **Directed**: DAG algorithms, topological sort
- **Undirected**: MST, bipartite checking

Graphs are powerful data structures that model complex relationships and are essential for solving many real-world problems efficiently.
