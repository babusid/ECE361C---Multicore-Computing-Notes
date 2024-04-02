- **Section 9.2:** Djikstra's algorithm
- **Section 9.3:** Convert shortest path problem to a lattice search problem and apply LLP
- **Section 9.4:** parallel technique for shortest path called ***delta-stepping***
- **Section 9.5:** Techniques for negative weights in the graph
- **Section 9.6:** Bellman-Ford algorithm
- **Section 9.7:** Algorithm for matrix multiplication, computing transitive closure of binary matrix
- **Section 9.8:** Extension of 9.7 to view it as a computation of shortest path for all vertex pairs.
## 9.2 Djikstra's Algorithm
- Djikstra's algorithm determines the shortest path from a given source node to all other nodes.
- Graph is defined as $(V,E,w)$:
	- $V -$ vertices 
	- $E -$ edges
	- $w-$ weights
- We assume the graph is loop free, and every vertex *x* has at least one incoming edge.
- ![[Screenshot 2024-04-01 at 5.19.36 PM.png]]
- This algorithm maintains an array $dist$
	- $dist[i]$  represents the ***cost*** to reach $v$ from $v_0$ .
	- Every element is initialized to $\infty$
	- When a vertex is initially discovered, $dist[i]$ becomes lesser than infinity
- In addition to $dist$, a boolean array $fixed$ is maintained
	- Every discovered vertex is either fixed or not
	- If a vertex is $fixed$ then that means the distance for that element cannot change any further. 
	- If a vertex is **not** $fixed$ then $dist[i]$ is the cost of the shortest path to *i* that goes through only fixed vertices
- The heap $H$ keeps all the discovered but non-fixed vertices as tuples of (*index, dist*)
	- Heap property applies to the dist field
- Algorithm at every step picks the closest unfixed node, fixes it, and then updates the dist values of all of its unfixed neighbors to the min of its existing value and the newly fixed node's distance + the distance between the two. It does not fix the neighbors.
	- If a node is discovered that is not in the heap, it is added 
- **Complexity Analysis:**
	- Insertion time / Deletion time from a heap takes $O(log(m)) = O(log(n))$ where *m* is the number of edges, *n* is number of vertices.
	- Overall time complexity is $O(m*log(n))$
- **Algorithm Steps**
	1. Set v 0 as the source vertex and assign it a distance of 0. Assign all other vertices a tentative distance of infinity. We insert the source vertex v 0 in the heap H. 2. Since the heap is not empty, we remove the vertex at the top of the heap. It is v 0 with a distance of 0. We mark that vertex as fixed. We examine its neighbors, v 1 and v 
	2.  For v 0 ! v 1 , the existing distance to v 1 is infinity. The distance 0 ( distance to v 0 ) + 9 (edge weight from v 0 to v 1 ) is less than infinity, so we update the distance to v 1 to 9 and add it to the heap. For v 0 ! v 2 , the existing distance to v 2 is infinity. The distance 0 + 2 is less than infinity, so update the distance to v 2 to 2 and add v 2 to the heap. 
	3. We remove the vertex at the top of the heap. The smallest distance among the unvisited vertices is 2 (for v 2 ), so set v 2 as the current vertex. We examine its neighbors, v 3 and v 
	4.  For v 2 ! v 3 , the existing distance to v 3 is infinity. Since 2 ( distance to v 2 ) + 6 (edge weight from v 2 to v 3 ) is less than infinity, we update the distance to v 3 to 8. For v 2 ! v 4 , the existing distance to v 4 is infinity. Since 2 + 5 is less than infinity, we update the distance to v 4 to 7.
## 9.3 Shortest path problem as an LLP lattice search
- In this section, we assume all edge weights are *strictly positive*.
- We want to find the minimum cost of a path from a **source** to all other vertices. 
	- Cost == sum of edge weights along path
- For any vertex $v$ let $pre(v)$ be the set of vertices $u$ such that $(u,v)$ is an edge in the graph
- Assume that every vertex $v$ has nonempty $pre(v)$ (except maybe the source vertex).
- **Lattice definition for search space**:
	- For each vertex $v_i$, we first assign $G[i] \in R_{\geq0}$  with the interpretation that $G[i]$ is the cost of reaching that vertex.
	- We call $G$ the *assignment* vector. 
	- The algorithm's invariant is as follows:
		- **For all $i$, the cost of any path from the source to $v_i$ is greather than or equal to $G[i]$**
		- The vector $G$ only gives lower bound on path cost, and there may not be any path to $v_i$ with cost $G[i]$
### Approach 1
- **Feasibility expression for assignment vector**
	- Feasibility for the state vector in this problem requires the notion of a parent
		- *similar to [[Chapter 8 - Basic Graph Algorithms]]*
		- We say that $v_i$ is **a** parent of $v_{j}$ in $G$  if **and only if** there is a direct edge $(v_i,v_j)$ **AND**  $G[j]$ is at least $(G[i] + w[i,j])$
			- $v_{i} \in parent(v_{j},G) \iff \exists (v_{i},v_{j})\in E \land G[j] \geq G[i] + w[i,j]$  
		- NOTE: a node could have more than 1 parent
	- **$G$ is a feasible assignment if and only if every node besides the source node has a parent**
		- $B_{path(G)} \equiv \forall j \neq 0 : (\exists i: parent(i,j,G))$
		- This means that the assignment is only feasible if you can go from any non-source node to the source node by following any parent edges.
- **Proof of lattice-linearity of feasibility expression**
	- Lemma: For any assignment vector $G$ that is not feasible, $\exists j: forbidden(G,j,B_path(G))$
	- Proof:
		- Suppose G not feasible
		- Then, there exists $j \neq 0$ such that $v_{j}$ doesn't have a parent
			- i.e. $\forall i \in pre(j), G[j] < G[i] + w[i,j]$
		- In this case, $forbidden(G,j,B_{path}(G))$ holds.
			- Pick any $H \geq G$. Suppose H is feasible.
			- Since for any $i \in pre(j), G[i] \leq H[i], G[j] < G[i] + w[i,j] \implies G[j] < H[i] + w[i,j]$ 
			- Whenever $H[j] = G[j], v_j$ does not have a parent
			- Therefore, $H$ not feasible
		-  Since H is not feasible if it contains the same element as G, this satisfies the definition of an appropriate forbidden state for lattice-linearity
- Since we know that $B_{path}$ is a Lattice-Linear Predicate, we can use the LLP algorithm with an appropriate **advance** function.
	- **In this case, the advance function is the following:** $min(G[i]+w[i,j] \vert i \in pre(j))$
		- Find the minimum cost out of all neighbors + traversal costs 
		- Similar to the all-neighbor scan in the [[Chapter 8 - Basic Graph Algorithms#8.6 Connected Components in an Undirected Graph Slow Algorithm | slow connected components algorithm]] from earlier.
### Approach 2
- For an unweighted graph, the above parallel LLP approach requires time equal to the distance of the farthest node from the root.
	- Sort of like [[Chapter 8 - Basic Graph Algorithms#8.5 Layering of a Directed Acyclic Graph|layering algorithm]]
- **Alternative feasible predicate**
	- This takes larger steps to go faster
	- First, we define a node $j$ to be $fixed$ in $G$ if either it is source or has a parent that is a fixed node
		- $fixed(j,G) \equiv (j=0) \lor (\exists j: parent(j,i,G) \land fixed(i,G))$ 
		- Borrowing fixed from [[#9.2 Djikstra's Algorithm|djikstra section]]
	- Source node is always fixed. Any node that can reach the source node using direct parent relation is also fixed.
	- Now we define the predicate for **feasible assignment:**
		- $B_{rooted}(G) \equiv \forall j: fixed(j,G)$
		- This is equivalent to the earlier predicate (i.e $B_{rooted}(G)$ iff $B_{path}(G)$)
		- **Proof of equivalence**
			- If $G$ satisfies our new predicate, then new every node other than the source has at least one parent by the definition of $fixed$. Hence, $B_{path}$ is also true. 
			- Conversely, suppose that every node except the source has a parent. Since parent edges can't be cyclic, by following the parent edges, we can go from any node to $v_0$.
	- Since the new predicate is equivalent, it is also lattice-linear.
	- As such, the threshold $\beta (G) = min_{(i,j): i \in pre(j)}(G[i]+w[i,j] \vert fixed(i,G), !fixed(j,G))$   is well-defined whenever the set of edges from the *fixed* to the *unfixed* edges is nonempty
		- ***This threshold is basically just saying "what is the closest unfixed node to a node that we know is reachable"***
	- If the set is empty, no *unfixed* vertex is reachable from the source. ***We call this set Heap***
		- *We use this set (as defined by the Beta threshold) for the forbidden state and the advancement operation*
	- **Now, we have to define the forbidden state and the advancement operation**
		- Suppose $!B_{rooted}(G)$ . Then $!fixed(j,G) \implies forbidden(G,j, B_{rooted}, \beta(G))$
			- **The forbidden state is not being fixed**
- **Algorithm**
	- In each iteration, we find the nodes that are not fixed (forbidden) and advance them. All nodes are advanced using both the **Beta** function defined earlier, as well as the **min operation from approach 1**
		- ie. combining $\beta (G)$ with $min(G[i] + w[i,j], i\in pre(j))$
		- to **combine** you just take the max of the two. 
			- $advance: G[j] := max\{\beta(G), min\{G[i] + w[i,j] \vert i \in pre(j)\}\}$
		- **What is this doing?**
			- First option, the beta function. Recall:
			  $\beta (G) = min_{(i,j): i \in pre(j)}(G[i]+w[i,j] \vert fixed(i,G), !fixed(j,G))$
				- This is saying that you increment $G[j]$ by an edge value that leads from a fixed i in your precursor set to your unfixed j. *This will automatically fix node j*
			- The second option is just linking to the closest element in your precursor set.
				- This may not fix your j
			- *Why is taking the max good here?*
				- #officehoursquestion 
	  - ![[Screenshot 2024-04-01 at 11.07.53 PM.png]]
	  - Same time complexity as Djikstra
## 9.4 Delta-Stepping
- 
## 9.5 Negative edges in a graph
- 
## 9.6 Bellman-Ford Algorithm
- 
## 9.7 Algorithm for matrix multiplication, computing transitive closures 
- 
## 9.8 Mapping 9.7 to computation of shortest path for vertex pairs


