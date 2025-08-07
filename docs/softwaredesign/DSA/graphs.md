# Graphs

## Shortest Path for Weighted Graph

## Dijkstra’s

`O((V + E) log V)` with a binary heap.

(Only works with pos weights)

[Video](https://youtu.be/EFg3u_E6eHU?si=KAFs7dVMTXp919W3)

Basic idea:

* Determine shortest path from src to dest
* To do this, determine shortest path to EVERY node along the way

Approach:

* Mark every node as distance=inf except src (0)

Main steps after:

* In map, update distances to neighbors (if shorter), mark as visited
* Next, travel to next shortest node away (use prio queue). Mark visited.
* Repeat
Use a visited set and skip in pq if already visited

EG:

```
      [A]
     /   \
   1/     \10
   /       \
 [B] ----1-- [C]
   \       /
   6\     /4
     \   /
      [D]
```

```python
import heapq

def dijkstra(graph, start):
    # graph is in the form: {node: [(neighbor, weight), ...], ...}
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    visited = set()

    # Priority queue: (distance_from_start, node)
    pq = [(0, start)]

    while pq:
        current_distance, current_node = heapq.heappop(pq)

        # Skip if already visited
        if current_node in visited:
            continue
        visited.add(current_node)

        # Relax edges
        for neighbor, weight in graph[current_node]:
            distance = current_distance + weight
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                heapq.heappush(pq, (distance, neighbor))

    return distances


# Example graph
graph = {
    'A': [('B', 1), ('C', 10)],
    'B': [('C', 1), ('D', 6)],
    'C': [('D', 4)],
    'D': []
}

print(dijkstra(graph, 'A'))
```

## Bellman-Ford Algorithm

Use if we have negative path weights.

DP strategy.

---

Intuition:
Bellman–Ford finds the shortest path from a source node by:

Trying all edges again and again (up to |V|−1 times), and updating a node’s distance whenever we find a cheaper way to get there through another node.

It's like saying:

“If I already know the shortest way to get to A, and there’s an edge from A → B, then maybe I now also know a better way to get to B.”

Because the longest possible simple path in a graph with V nodes has V−1 edges — so if we relax all edges V−1 times, we’re guaranteed to propagate all minimal costs forward.

---

[Bellman Ford](https://youtu.be/FtN3BYH2Zes?si=ady6gKsfmVi5RB88)

`O(V * E)`

Theorem 1: In a "graph with no negative-weight cycles? with N vertices, the shortest path between any two vertices has at most N-1 edges.

A negative weight cycle is one that results in a negative path after the cycle is completed.

For example, a positive weight cycle could be

```
A - 1 -> C - 2 -> B - -1 -> A
```

![Neg weights](img/graphs/negative-weight-cycles.png)


The cost from A to B is +2, going back to A we subtract one resulting in a weight greater than zero (2).

The reason the shortest path has at most N-1 edges is that because there are no negative weight cycles, starting over from the original cycle we'll just increase the cost or make it the same.

So, basic idea - "relax" edges N-1 times (N is num vertices).

Relaxation refers to trying a connection and seeing if it reduces the cost to get to that vertex. If it does we reduce it accordingly and that is relaxation

Initially mark the cost to get to a vertex as Infinity for all vertex
Loop over all edges in any order.

For an edge between vertex u and v.

```
if dist[u] + cost(u,v) < dist[v]:
    dist[v] = dist[u] + cost(u,v)
```

Full example:

```python
def bellman_ford(n, edges, source):
    INF = float('inf')
    dist = [INF] * n
    dist[source] = 0

    # Relax all edges (V - 1) times
    for _ in range(n - 1):
        for u, v, w in edges:
            if dist[u] != INF and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    # Optional: Detect negative-weight cycle
    for u, v, w in edges:
        if dist[u] != INF and dist[u] + w < dist[v]:
            raise ValueError("Graph contains a negative-weight cycle")

    return dist

# Graph:
# A → B (4)
# A → C (2)
# C → B (1)
# B → D (2)
# C → D (5)

# Convert node names to indices: A=0, B=1, C=2, D=3
edges = [
    (0, 1, 4),  # A → B
    (0, 2, 2),  # A → C
    (2, 1, 1),  # C → B
    (1, 3, 2),  # B → D
    (2, 3, 5),  # C → D
]

n = 4
source = 0  # A

distances = bellman_ford(n, edges, source)

print("Shortest distances from A:")
print(f"A: {distances[0]}")
print(f"B: {distances[1]}")
print(f"C: {distances[2]}")
print(f"D: {distances[3]}")
```

Note:

We can limit the maximum number of edges traversed to see the shortest path with an additional restriction of how many edges we can traverse. For example if we wanted two Traverse a maximum of K edges we would use that instead of N-1

If we do this we must use a temp variable
