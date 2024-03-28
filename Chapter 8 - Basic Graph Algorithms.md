## 3-19-24 Lecture

- 3 major tasks in graphs that we care about
	1. Traversal
	2. Layering of an acyclic graph
	3. Finding connected components in an undirected graph

- In the parallel setting, directed graphs are **harder** than **undirected** graphs.

### General Traversal in a graph
- Given a graph (V,E) and a source vertex V0, our goal is to find all the vertices reachable from that source vertex
- G[i] =  1 if Vi is reachable directly from V0
	- Vi is another node in the graph
- **How do we find all the nodes that are reachable in a parallel manner?**
	- First, we need a predicate:
		- B_traverse = G[0] AND (all(Vi,Vj) in E, G[j]>=G[i])
		- G is a boolean array such that Vi is reachable from V0 if G[i] is true (or 1)
	- This predicate requires V0 to be reachable from V0 and **for all edges** (Vi, Vj), if Vi is reachable, then so is Vj
		- TLDR: Source is reachable from itself, and reachability is transitive
	- This predicate is a conjunctive predicate, and the first conjunct is a local predicate. To prove lattice-linearity, we need to show that the second conjunct is also lattice linear.
		- If second conjunct is false, then there exists an edge (Vi, Vj) in E such that G[i] is 1 and G[j] is 0. In this case, unless G[j] is set to 1, the predicate cannot be true, indicating that this is a forbidden state. This leads to the following algorithm
	- ![[Screenshot 2024-03-19 at 2.12.51 PM.png]]
- The LLP algorithm for graph traversal is general and non-deterministic. **Given certain strategies for computing forbidden indices, it can implement breadth first or depth first search.**
### Breadth First Search Tree
#### LLP approach
- Previous section found the set of reachable vertices from a given vertex
- Suppose you instead want to find the BFS tree rooted at any given node.
- A vertex j is defined to be forbidden if it has a predecessor i such that G[j] > G[i] + 1. If none of the vertices is forbidden, we get 
	- $\forall j \neq 0: \forall(i,j) \in E: G[j] \leq G[i]+1$
- ![[Pasted image 20240319141924.png]]
- Algorithm terminates when there is no forbidden vertex
#### More traditional approach (BFS-Traversal-Parallel)
- ![[Pasted image 20240319142214.png]]
- Let N be the number of vertices
- Let M be the number of edges
- In sequential setting, BFS is **O(M+N)**
- In parallel setting (with BFSTP)
	- Forall iteration constructs one level of the BFS tree at a time
	- Therefore, it is O(height of the BFS tree)

### DFS Tree in a Directed Graph
- DFS is very hard to parallelize effectively
- Only went over sequential version in class
#### Topological Sort of DAG (Directed Acyclic Graph)
- Example of Topological sort is sorting classes at UT by pre-req to find earliest semester you can take the course
- In predicate form, topological sort is as follows:
	- Determine the least vector G that satisfies: $B= \forall(i,j) \in E: G[i] < G[j]$
	- This predicate is lattice linear because:
		- If it is false, there exists an edge leading from i to j where i is further from the source than j
		- This is lattice-linear because if it is false, it can never become true without advancing the state / changing something.
- ![[Pasted image 20240319143948.png]]


#### Connected Components in an Undirected Graph
- **REVIEW TEXTBOOK**
- Note: this was the most confusing part, i got lost very quickly
- Problem is finding the largest subgraph
- Key to doing this in a fast way is to utilize **pointer jumping.**
	- However, pointer jumping previously was only in a directed context
	- Recall that pointer jumping was the idea that instead of pointing to your parent, why not just point to your grandparent?
- ![[Screenshot 2024-03-19 at 3.02.38 PM.png]]