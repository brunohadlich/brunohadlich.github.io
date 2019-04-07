---
layout: post
title: Python implementation of Dijkstra algorihthm
---

When I was trying to implement Dijkstra algorithm in Python from pseudocodes I found on websites like this one from Wikipedia.

``` text
1  function Dijkstra(Graph, source):
2      dist[source] ← 0                           // Initialization
3
4      create vertex set Q
5
6      for each vertex v in Graph:           
7          if v ≠ source
8              dist[v] ← INFINITY                 // Unknown distance from source to v
9          prev[v] ← UNDEFINED                    // Predecessor of v
10
11         Q.add_with_priority(v, dist[v])
12
13
14     while Q is not empty:                      // The main loop
15         u ← Q.extract_min()                    // Remove and return best vertex
16         for each neighbor v of u:              // only v that are still in Q
17             alt ← dist[u] + length(u, v) 
18             if alt < dist[v]
19                 dist[v] ← alt
20                 prev[v] ← u
21                 Q.decrease_priority(v, alt)
22
23     return dist, prev
```

I run into a problem with Priority Queues in which a function called decrease_priority is called, the problem is that Python does
not have a default library with a priority queue implementation that provides such feature, so to work around this I create a
list that will keep track of vertexes that have already been traveled, in the example below it is "vertex_in_priority_queue",
in each iteration I check if the element that was popped from the queue "priority_queue_sorted_by_distance" has been traveled
in "vertex_in_priority_queue", if it has I simply ignore it and go to the next iteration, if not I mark it as traveled and make
all the process described between lines 16 and 21 of the pseudocode. The code below was executed with Python 3.

``` python
import math, heapq

def dijkstra(vertexes, edges, source_vertex, destination_vertex, is_direct_graph = False):
    distance_from_source_to_vertexes = {}
    parent_vertexes = {}
    priority_queue_sorted_by_distance = []
    distance_from_source_to_vertexes[source_vertex] = 0
    vertex_in_priority_queue = {}
    edges_weights = {}

    for vertex in vertexes:
        if vertex != source_vertex:
            distance_from_source_to_vertexes[vertex] = math.inf
        parent_vertexes[vertex] = ''
        heapq.heappush(priority_queue_sorted_by_distance, (distance_from_source_to_vertexes[vertex], vertex))
        vertex_in_priority_queue[vertex] = True
        edges_weights[vertex] = []

    for edge in edges:
        edges_weights[edge[0]].append((edge[1], edge[2]))

        if not is_direct_graph:
            edges_weights[edge[1]].append((edge[0], edge[2]))

    while priority_queue_sorted_by_distance:
        vertex_lowest_distance = heapq.heappop(priority_queue_sorted_by_distance)[1]
        if vertex_in_priority_queue[vertex_lowest_distance]:
            vertex_in_priority_queue[vertex_lowest_distance] = False
            for neighbor in edges_weights[vertex_lowest_distance]:
                total_distance = distance_from_source_to_vertexes[vertex_lowest_distance] + neighbor[1]
                if total_distance < distance_from_source_to_vertexes[neighbor[0]]:
                    distance_from_source_to_vertexes[neighbor[0]] = total_distance
                    parent_vertexes[neighbor[0]] = vertex_lowest_distance
                    heapq.heappush(priority_queue_sorted_by_distance, (distance_from_source_to_vertexes[neighbor[0]], neighbor[0]))

    return distance_from_source_to_vertexes, parent_vertexes

if __name__ == '__main__':
    vertexes = ['S', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M']
    edges = [('S', 'A', 7), ('S', 'B', 2), ('A', 'B', 3), ('A', 'D', 4), ('D', 'B', 4), ('D', 'F', 5), ('B', 'H', 1), ('H', 'F', 3), ('H', 'G', 2), ('G', 'E', 2), ('E', 'K', 5), ('K', 'I', 4), ('K', 'J', 4), ('I', 'J', 6), ('I', 'L', 4), ('J', 'L', 4), ('L', 'C', 2), ('C', 'S', 3), ('G', 'M', 1), ('F', 'M', 1), ('K', 'M', 1)]
    source_vertex = 'M'
    destination_vertex = 'S'
    (distance_from_source_to_vertexes, parent_vertexes) = dijkstra(vertexes, edges, source_vertex, destination_vertex)
    print('Source vertex: ' + source_vertex)
    print('Destination vertex: ' + destination_vertex)
    print('Lowest distance from ' + source_vertex + ' to ' + destination_vertex + ': ' + str(distance_from_source_to_vertexes[destination_vertex]))
    print('Path from ' + source_vertex + ' to ' + destination_vertex + ': ', end = '')
    current_vertex = destination_vertex
    path = []
    while current_vertex != '':
        path.append(current_vertex)
        current_vertex = parent_vertexes[current_vertex]
    while len(path) > 1:
        print(path.pop(), end = ' -> ')
    print(path.pop())
    for vertex in distance_from_source_to_vertexes:
        print('Lowest distance from ' + source_vertex + ' to ' + vertex + ': ' + str(distance_from_source_to_vertexes[vertex]))
    for vertex in parent_vertexes:
        print('Parent of vertex ' + vertex + ': ' + parent_vertexes[vertex])
```
