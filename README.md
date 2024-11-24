In this lab, I solved the $n^2 - 1$ puzzle problem by interpreting it as a pathfinding problem. I can adopt this perspective because the puzzle can be seen as a problem requiring a sequence of decisions, where each decision corresponds to the move needed at each step to transition from the puzzle's initial random configuration to the desired final configuration.  
The pathfinding algorithm I considered most effective to be implemented for solving this specific problem is the A* algorithm.  
I decided to use a graph search approach and chose the Manhattan distance as the admissible and consistent heuristic.  
The Manhattan distance is a heuristic commonly used in pathfinding and optimization problems, it measures the total distance a tile needs to travel to reach its goal position assuming movement is restricted to a grid and only allowed in horizontal or vertical directions.  
It's admissible because it never overestimates the true cost to solve the puzzle as it assumes no obstructions or collisions and satisfies the triangle inequality ensuring that moving from one state to another does not violate heuristic consistency.  

In pathfinding algorithm A*, the use of a heuristic function plays a crucial role in guiding the search toward the goal state more efficiently.   
Unlike uninformed search algorithms (for example, Breadth-First Search or Uniform Cost Search), A* leverages heuristic information to prioritize exploration of the most promising states, thus reducing the number of expanded nodes and accelerating convergence.  
A* evaluates each state 𝑛 using the formula $f(n) = g(n) + h(n)$
where:
- $g(n)$ is the cost to reach the current state from the start state.
- $h(n)$ is the heuristic estimate of the cost to reach the goal from the current state.

The heuristic function $h(n)$ provides an estimate of how "close" the current state is to the goal.  
By including $h(n)$, A* can focus on states that are likely part of the optimal path and reduce the exploration of irrelevant or less promising states.  
Without a heuristic A* behaves like Uniform Cost Search expanding nodes based solely on $g(n)$, which represents the total cost from the start.  
This leads to unnecessary exploration of states that are far from the goal but have a low accumulated cost.  
With a heuristic like Manhattan distance, A* expands nodes by balancing the cost $g(n)$ with the estimate $h(n)$: this means that states closer to the goal (based on $h(n)$) are prioritized, directing the search toward the solution.  
An admissible heuristic ensures that $h(n) ≤ h(n)^*$ where $h(n)^*$ is the true cost to the goal.  
This guarantees that A* never ignores the optimal solution but still avoids exploring unnecessarily.  
Moreover, a consistent heuristic satisfies $h(n) ≤ c(n,n') + h(n')$ where $c(n,n')$ is the cost to transition from n to n': this ensures that the estimated cost f(n) along any path is non-decreasing, preventing redundant revisits of states and further improving efficiency.

### Graph search
In graph search, it is essential to maintain a set of already expanded nodes to ensure that they are not re-added to the priority queue and then re-examined.  
This is because, unlike tree search, graph search acknowledges that multiple paths can lead to the same state.  
Without a list of expanded nodes, the same state might be generated multiple times via different paths and added to the priority queue repeatedly.  
This would lead to unnecessary expansions increasing computational overhead and wasting resources.  
By tracking expanded nodes, the algorithm avoids revisiting states, reducing the total number of nodes expanded.  
This is particularly critical in problems with cyclic or densely connected graphs: in a graph with cycles, failing to track expanded nodes could lead to infinite loops, where the algorithm keeps revisiting the same states.

### Mutable objects can't be keys in a set
For this purpose, using `bytes` and `tobytes()` to convert a NumPy array into an immutable object is essential for efficiently tracking explored/expanded states in graph search.  
Python's set and dict rely on hashing for fast lookups and require their keys to be immutable.  
A NumPy array is mutable meaning its content can be changed after creation.  
Because of this mutability, NumPy arrays cannot directly be used as keys in a set or dict.  
`bytes` is an immutable sequence of bytes in Python: once created, its content cannot be changed.  
By converting a NumPy array to bytes using `tobytes()`, I create an immutable representation of the array that retains the exact data and layout of the array in memory and can be hashed, allowing it to be used as a key in a set or dict.  
Hashing is used to quickly check for membership in a set or as a key in a dict.  
The hash function calculates a fixed-length value (hash) from the object which serves as its "identifier."  
The object must not change after its hash is calculated because, if the object changes, its hash would change, leading to inconsistencies.
By converting the array to `bytes`, I ensure the object remains immutable.  
In addition, objects with different content should produce different hashes.
The bytes representation from `tobytes()` ensures that arrays with different values produce distinct hashes.

### Conditions for Solvability
The  $n^2 - 1$ puzzle has a fundamental mathematical property: not all initial configurations can lead to the goal state.  
This limitation arises from the structure and rules of the puzzle, which restrict the ways tiles can be rearranged.  
The puzzle can be represented as a permutation of tiles with one empty space (0) allowing movement. 
The solvability of a configuration depends on the number of inversions in the arrangement.
An inversion occurs when a higher-numbered tile precedes a lower-numbered tile in the linearized array of the puzzle's tiles (ignoring the empty space).  
By performing some mathematical analyses, I realized that the rules for determining whether a configuration is solvable differ depending on the dimension **n** of the puzzle:
- **Case 1: n odd**  
  A configuration is solvable if the number of inversions is even.
- **Case 2: n even**  
  A configuration is solvable if:  
  1. The number of inversions is even and the blank tile (0) is on an odd row from the bottom.  
  2. The number of inversions is odd and the blank tile (0) is on an even row from the bottom.


To ensure that my algorithm starts with a solvable configuration, I developed the function generate_random_solvable_puzzle().  
It ensures solvability in the following way:
- It starts from the goal state since the goal state is guaranteed to be solvable.
- Apply valid moves: the function performs a series of random moves from the goal state using the puzzle's rules to generate new configurations.
Since every move is valid and starts from a solvable state, the resulting configuration remains solvable.

Therefore, by design, I eliminate the need for computationally verifying the solvability of the resulting configuration, as it is guaranteed.

### Time Complexity: `heapq` vs  list

Finally, discussing the computational efficiency of the operations, I provide an in-depth explanation of why it is advantageous to use the heapq data structure to implement the priority queue instead of a list since using `heapq` to implement a priority queue offers significant computational advantages compared to a basic list.  
`heapq` is a Python module that implements a binary heap, a specialized tree-based data structure:
- It maintains the heap property: the smallest (or largest, for a max-heap) element is always at the root.
- Operations like insertion and extraction of the smallest element are highly efficient.

In the context of A*, the priority queue needs to quickly retrieve the node with the lowest f(n) and efficiently add new nodes as they are generated.  
The key operations of a priority pueue are:
- Insert (Push): add a new element to the queue.
Must maintain the order of priorities for efficient extraction of the minimum.
- Extract Minimum (Pop): remove and return the element with the smallest priority.
The queue must always keep track of the smallest element efficiently.
- Peek: view the smallest element without removing it.

The comparison table shows the time complexities for these operations when using:
1. A **binary heap** (`heapq`)
2. A **list** (sorted or unsorted)

| **Operation**         | **heapq** (Binary heap) | **list** (sorted or unsorted)          |
|------------------------|---------------------------|-----------------------------------------|
| Insert                 | O(log n)                 | O(1) (unsorted) or O(n) (sorted)       |
| Extract Minimum        | O(log n)                 | O(n) (unsorted) or O(1) (sorted)       |
| Peek                   | O(1)                     | O(n) (unsorted) or O(1) (sorted)       |

### Experimental results

In conclusion, I report the number of moves and the number of states explored by my implementation of the A* algorithm to solve 3 instances of different sizes of the problem, each with 3 randomly generated starting configurations of the puzzle.  
Each of the 3 execution durations corresponds to the use of the Intel(R) Core(TM) i9-14900KF processor:  
| Puzzle Size | Starting State                                      | Number of Moves | Number of Explored Nodes | Execution Time (hh:mm:ss.microsecond) |
|-------------|----------------------------------------------------|-----------------|--------------------------|---------------------------------------|
| 3           | `[[4, 1, 3],`                                      | 9               | 17                       | 00:00:00                              |
|             | `[7, 2, 0],`                                       |                 |                          |                                       |
|             | `[5, 8, 6]]`                                       |                 |                          |                                       |
| 4           | `[[1, 2, 11, 7],`                                  | 23              | 128                      | 00:00:00.006000                       |
|             | `[0, 5, 12, 3],`                                   |                 |                          |                                       |
|             | `[9, 6, 15, 4],`                                   |                 |                          |                                       |
|             | `[13, 10, 8, 14]]`                                 |                 |                          |                                       |
| 5           | `[[1, 2, 3, 9, 4],`                                | 29              | 898                      | 00:00:00.071051                       |
|             | `[6, 7, 8, 15, 5],`                                |                 |                          |                                       |
|             | `[16, 11, 12, 13, 10],`                            |                 |                          |                                       |
|             | `[17, 14, 23, 22, 24],`                            |                 |                          |                                       |
|             | `[21, 0, 18, 20, 19]]`                             |                 |                          |                                       |


