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

`O(V * E)`
