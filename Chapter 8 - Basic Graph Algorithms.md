# 3-19-24 Lecture

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

# TextBook Notes
## 8.1 Introduction
- Graph theory is extremely widespread
- Graphs can be *undirected* or *directed*
- Graphs can be *simple* or *complex*
	- *simple* means no loops or parallel edges
- Graphs can be *weighted* or *unweighted*
- Graphs can be *acyclic* or not
- TLDR: Graphs have many properties
- **In this chapter:**
	- **Section 8.2:** parallel LLP algorithm to traverse a graph
	- **Section 8.3:** parallel LLP algorithm to construct a ***BFS tree*** from a graph given a source vertex.
	- **Section 8.5:** gives a parallel LLP algorithm to topologically sort a *directed, acyclic* graph (DAG)
	- **Section 8.6:** gives an LLP algorithm to find connected components in an undirected graph
	- **Section 8.7:** gives a faster algorithm based on [[Chapter 7 - List Ranking and Pointer Jumping Algorithms#7.2 Pointer Jumping Algorithm| pointer jumping]].

## 8.2 General Graph Traversal using LLP

- Given a graph $(V,E)$ and a source vertex $v_0$, **our goal is to find all vertices that are reachable** from $v_0$.
	- $V$ is the vertex set, $E$ is the edge set.
- This problem can equivalently be viewed as finding a vertex set $W \subseteq V$  such that $v_{0} \in W$ and $\forall x,y : (x \in W) \land (x,y) \in E \implies y\in W$ 
	- Basically just saying mathematically what we've already said. Any vertex that has an edge to the subset becomes part of the subset. *this lends itself to an iterative expansion of sorts?*
- We model $W$ using a boolean array $G$ such that $v_{i}\in W$ if and **only if** $G[i] = True$.
- This leads us to the **predicate** we need to detect:
	- $B_{traversable} = G[0] \land (\forall (v_{i},v_{j}) \in E : G[i] \leq G[j ])$
		- What is this saying?
			1. $G[0]$ must be true. This is obvious because $G[0] \equiv v_0$
			2. For every edge in the edge set, if one of the nodes is reachable, so must be the other one.
	- *Book says this is a conjunctive predicate, and the first conjunct is a local predicate. What does that mean?*
		- #officehoursquestion 
- This predicate leads us to the **forbidden state**:
	- $forbidden(0,G) = (G[0]==0)$
	- $forbidden(i,G) = (\exists (v_{i}, v_{j}) \in E: G[i] > G[j])$
- **Proof of lattice-linearity:**
	- The first conjunct is local to one element, meaning that it is lattice-linear
		- Verify this
	- We have to verify that the second conjunct is also lattice-linear.
		- If the second conjunct is false, then there exists some edge such that $G[i] = 1, G[j] = 0$. In this case, unless $G[j]$ is also set to 1, the predicate can **never become true.**
		- As such, this conjunct is also lattice-linear
- **Algorithm:**
	- Initialize $G[0]=1$
	- Initialize $G[1 ... n] = 0$ 
	- $forbidden(j): \exists i: j \in edges(i), (G[i]=1) \land (G[j] = 0)$
	- $advance(j): G[j] = 1$
	- ![[Screenshot 2024-04-01 at 11.05.21 AM.png]]
- The algorithm is non-deterministic, and various strategies could be used to compute and select the forbidden indices and the order in which you process them.
- We now discuss a particularly common strategy 
###  Frontier vertex selection strategy.
- This is one of the strategies that you could use to select forbidden indices to process and in which order
- You maintain a set $S$ of *frontier* vertices - a set that is reachable, but their outgoing edges have not been explored yet.
	- Recall my remark earlier about "iterative expansion". Think of this frontier set like the border of the reachable subset at any point.
- The algorithm **Traversal-Sequential*** gives a sequential implementation of the LLP algorithm using the frontier set concept
- By using various strategies for *selecting* an element out of $S$, we traverse the graph in ***different orders.***
	- If the frontier set is held in a *queue* data structure (FIFO), where removal (*selection*) from $S$ corresponds to a "pop", and addition corresponds to a "push", then the LLP algorithm breaks down into a **breadth-first search.**
	- If the frontier set is held in a *stack* data structure (LIFO), where selection and addition correspond similarly, then the LLP algorithm breaks down into a **depth-first search.**
	- You could use *whatever data structure you want to use* with ==whatever push/pop operation you want.==
		- It will simply correspond to a different traversal style.

## 8.3 Breadth first search *tree* in a directed graph

- In the previous section we saw an LLP algorithm for traversal, which when combined with the correct strategy, resulted in a **breadth-first traversal** to find the set of reachable nodes from a given source.
- Suppose now that instead, **we want to compute a *breadth-first search tree*** rooted at some source.
- We want to do this using LLP
#### LLP-Traversal-BFS:
- **Lattice Definition:**
	- First, we need a lattice if we want to use LLP
	- We define the lattice for this problem to be **the set of distance vectors** where in each vector, $G[i]$ represents the distance of vertex $i$ from $v_0$
- **State vector G**
	- Initially, we set $G[0] = 0, G[1 ... n] = \infty$
		- 0 is the source vertex
		- The rest are unknown when we start so we set it to infinity
	- To be a feasible distance vector, it has to obey one major constraint:
		- For every connected vertex pair that is reachable from the root, they will either be equidistant from the root, or one will be further away. However, the "further away" one cannot be more than 1 unit further, because there is a direct link connecting the closer one to the further one. 
		- The predicate is formally defined below.
- **Formally, the predicate is defined as:** 
	- $B(g) = \forall j \neq 0: \forall(i,j) \in E: G[j] \leq G[i]+1$
- **Our goal is to maximize G, subject to BFS**
	- We want to maximize G because maximizing the vector means we have larger elements present. Larger elements mean we traversed further?
- **Forbidden state:**
	- $forbidden(G,j) = \exists i \in parents(j): G[j] > G[i]+1$
	- $parents(j)$ is the set of all precursor connected to $j$. For this algorithm, you assume you know the graph topology ahead of time.
	- This is effectively saying, you cannot be more than 1 away from your lowest direct neighbor
	- To **advance** out of the forbidden state, you simply update to be only one away from your lowest neighbor
- **Algorithm:**
	1. Initialize $G[0] = 0, G[1 ... n] = \infty$  
	2. $forbidden(G,j) : \exists i \in parents(j): G[j] > G[i]+1$
	3. $advance(G,j) : G[j] = G[i]+1$
- Algorithm terminates when there are no forbidden vertices
- Any vertex that ends with infinity is not reachable
- ***This algorithm relies on knowing the precursor (parents) of each vertex ahead of time. Alternatively, you can use the adjacency list for each vertex, if this is not feasible.***
![[Screenshot 2024-04-01 at 11.44.20 AM.png]]
#### BFS-Traversal-Parallel
- This approach is ***not llp***
- This approach **also** does not need to know precursor information / topology ahead of time. May be better for very large graphs that cannot fit into memory
- Key is to maintain a *parent* pointer variable that gives the parent of any node in the graph. 
- If a node $x$ is reachable from $v_0$ then by following $parent$ from $x$ to $v_0$ you get the shortest path from $v_0$ to $x$. 
- Set $S$ keeps the set of reachable indices.
- ![[Screenshot 2024-04-01 at 11.55.14 AM.png]]


## 8.4 Depth - First Search Tree in a Directed Graph
- **DFS is really hard to parallelize**
- DFS-Traversal algorithm visits all vertices reachable from a source vertex in a depth first manner
- Each node has an element in a global vector $G$ indicating whether or not it is reachable.
- The variable $S$ is used ass a stack to store vertices while traversing the graph. 
- The $discovered$ variable is used to store "when" a vertex is explored
- The $finished$ variable is used to store the time when the algorithm finishes
- The $parent$ variable is used to store the tree. Anytime a vertex is visited for the first time from another vertex, that source vertex is stored as its parent
- The $discovered, finished$ variables are only used for recording the time (tick) when the vertex is started to be explored and finished for exploration. We don't use them for anything else.
![[Screenshot 2024-04-01 at 12.21.45 PM.png]]

## 8.5 Layering of a Directed Acyclic Graph
- Directed Acyclic graphs can be "ordered" or "layered" such that every vertex $i$ in the graph is assigned a number $G[i]$ such that if there is an edge $(i,j)$ the number assigned to $i$ is strictly less than the number assigned to $j$.
- **A good example of this is a graph of courses available at a university linked by prerequisite courses, with integers corresponding to the earliest semester in which a course can be taken by a student.**
- In this case, the search lattice consists of vectors of natural numbers
	- Each course gets assigned some natural number indicating when it can be taken, each course has an index in the vector, the vector represents the ordering
- We want to determine the least vector $G$ that satisfies the constraint of topological ordering
	- Least is good because it minimizes the distance between starting at university and taking a class, which is helpful.
- The constraint of topological ordering can be represented as a lattice-linear predicate in the following way:
	- $B_{layer(G)}= \forall (i,j) \in E(G),G[i] < G[j]$
	- $forbidden(G,i) = \exists (i,j) \in E(G), G[i] \geq G[j]$
	- *For simplicity we define $fixed(j) \equiv \forall (i,j) \in E(G),G[i] < G[j]$*
- **Proof of predicate lattice-linearity:**
	- If $B_{layer}$ is false, that implies that there exists some (i,j) pair where $G[i] \geq G[j]$. The predicate cannot be false without this condition, which is the definition of the forbidden state. Unless $G[j]$ is advanced to become not forbidden, the predicate can never become true. Therefore, **since the predicate can only become true if the forbidden state is eliminated**, we know the predicate is lattice-linear
- **Algorithm:**
	- Initialization: $\forall j \in V(G), G[j] = 0$
	- Initialization: $\forall j \in V(G), pre(j) = \vec{0}, fixed(j) = True$
		- For all of the nodes that have no precursors in the graph topology with respect to the given source node, set their fixed attribute to true (ie. the predicate is already true for them, they won't be forbidden)
	- $Forbidden(G,j): !fixed[j] \land \forall i \in pre(j): fixed[i]$
		- All nodes where all of their precursors are finished are forbidden from being unfinished.
		- This is to propagate outwards layer by layer
	- $advance(G,j): G[j] = max(G[\forall i \in pre(j)])+1, fixed[j] = True$ 
		- Take the maximum of all your precursors / the vertices in the layer before, and add 1 to set your own value. Then set your fixed to true, so that eventually the algorithm goes to the next layer
- This algorithm takes **parallel time** equal to the length of the longest path in the DAG (because it goes layer by layer, and each step on the longest path defines a new layer)
- The **work complexity is O(n+m)**
- *Book says come up with your own topological sort, but is this not a topo sort?*
	- No, this is just a layering algorithm to split the vertices into layers.
	- Topo sort is a **total ordering on all of the *vertices*** that maintains the constraint of $\forall (i,j) \in E(G),G[i] < G[j]$
## 8.6 Connected Components in an Undirected Graph: Slow Algorithm

- We are given an undirected graph $(V, E)$ on $n$ vertices numbered $1 ... n$
- **Objective: label each vertex $i$ with $G[i]$ such that $G[i]$ is the largest numbered vertex *in the component to which i belongs.*** 
	- We are trying to find connected components of graphs / subgraphs
	- To do this, each connected component is given the identity of the "largest" vertex in its graph, which is indicated by the G value for each vertex
	- We want every vertex in that component to take on that identity, so think sort of like "heat" you want the value to propagate through and even out in that component
- **State Vector / Predicate:**
	- We are going to use an LLP algorithm, so we need to define what makes a feasible state vector.
	- A vector $G$ is feasible if it obeys:
		- $B_{connected-components} = (\forall i: G[i] \geq i) \land (\forall(i,j) \in E : G[i] \geq G[j])$
			1. For every vertex, its G value is equal to or greater than its index
				- In a completely unconnected graph, each vertex is just labeled by its index, and you have $n$ components
			2. For every pair of connected vertices, one of the vertices must be greater than or equal to the other
				- However, note that this is an undirected graph. As such, if $(i,j) \in E \implies (j,i) \in E$
				- Therefore, we're really just trying to make the connected vertices have the same label
- **Proof of lattice-linearity**
	1. The first conjunct is lattice linear **because it is a conjunction of wholly local predicates** (each vertex)
	2. The second conjunct is lattice-linear because if there ever exists an edge $(i,j)$ such that $G[i] < G[j]$ the predicate cannot become true unless $G[i]$ is advanced to $G[j]$ (ie. advancing out of the forbidden state)
- **Algorithm:**
	- Initialization: $\forall j \in V, G[j] = j$
	- $Forbidden: (\exists j \in V \land (i,j) \in E: G[i]<G[j])$ 
	- $Advance(i,j): G[i] = G[j]$
	  ![[Screenshot 2024-04-01 at 2.10.28 PM.png]]
- **Discussion:**
	- $n$ threads in the algorithm, one for each vertex, responsible solely for initializing and updating that vertex's G value.
	- If any thread finds that there exists another vertex where that vertex and its vertex are adjacent, and its own G value is less, then it updates / runs advance
	- Worst case time complexity is $O(n)$ in a synchronous model. The graph may be a **linked list** and if each thread runs sequentially, it will take $O(n)$ time to propagate all the way down.
## 8.7 Connected Components in an Undirected Graph: Fast Algorithm
- **Overview:**
	- The prior algorithm was LLP, but had a worst case time complexity of $O(n)$. We saw in [[Chapter 7 - List Ranking and Pointer Jumping Algorithms#7.2 Pointer Jumping Algorithm | the previous chapter]] that pointer jumping is a highly effective technique to make algorithms run in sub-linear, logarithmic time. **This section details applying it to the connected components problem.** 
	- Other applications of pointer jumping [[Chapter 7 - List Ranking and Pointer Jumping Algorithms#7.3 Applications of the Pointer Jumping technique|here]]
- In this algorithm, we use the notion of a ***prioritized forbidden function.***
	- There could be many reasons for a index to be forbidden.
	- Computing all of these reasons could be expensive and hard
	- Idea of *PFF* is to front-load the computation of the *easier* reasons (forbidden functions)
- In the previous algorithm, the forbidden function requires:
	1. A node to scan all of its adjacent neighbors
	2. Ensuring that it is the same value as its adjacent neighbors
- **Scanning all of the adjacent neighbors is a difficult task.**
- If we view $G[i]$ as the **parent of $i$** it is *clear* that $G[i]$ should be greater than or equal to $G[G[i]]$
	- Parent means there is a connection. If you are connected to a parent, you are connected through them to a ***grandparent*** as well. 
	- If we want all adjacent nodes to be marked the same, a node will be marked identically to it's parent, but ***also to its grandparent.***
- **Condensing the graph to link nodes to their grandparents "recursively" is very easy compared to scanning all neighbors periodically. If we do this upfront, we can make the algorithm much faster**
- Instead of using **G** as our state vector, let **parent** be the state vector for this algorithm.
- Then, condensing the graph can be done by:![[Screenshot 2024-04-01 at 2.27.37 PM.png]]
	- This will turn any component into the graph, irrespective of starting topology, into a star network with one root and $componentsize - 1$ leaves.
- This **runs in $O(log(n))$** because it halves the height of the tree in each iteration
- **Combining this condensing approach with the second conjunct of the predicate from earlier yields:**![[Screenshot 2024-04-01 at 2.37.42 PM.png]]
	- You can either make the condensing part of the forbidden function in an LLP solver, or you can condense ahead of time. Even if it is part of the forbidden function, it removes the worst-case $O(n)$ scan that happens in the slower implementation.
- This particular implementation will execute the first ensure until it is globally true, which will take $O(log(n))$ time. Once that is done, the graph will be a collection of rooted stars.
- The **second ensure statement merges the rooted stars that touch.**
	- You may have nodes belonging to multiple rooted stars based on how you define parents of nodes
	- If there are no touching rooted stars (edges spanning them) then we are done, and every rooted star is a component on its own.
- Merging has its own complexity associated with it
- We refer to rooted stars as *supervertices.*
	- Two vertices are in the same supervertex if they have the same parent
- If there is any edge from a rooted star *i* to a rooted star *j*, there are three cases
	1. $parent[i] < parent[j]$ so star *i* joins star *j*
	2. $parent[j] < parent[i]$ so star *j* joins star *i*
	3. $parent[j] < parent[i]$ BUT $parent[i] < parent[k]$ AND *j* is aware of *k*, *j* joins star *k*. Eventually, *i* will join *k* on a later pass
- Whenever rooted stars are being merged, we get rooted trees (1-level), but we can reapply pointer-jumping. Algorithm terminates when no rooted stars are left as merge candidates.
![[Screenshot 2024-04-01 at 2.59.09 PM.png]]
- ==Since there are two applications of pointer jumping, one nested within the other, the overall time complexity os $O(log(log(n)))$ or $O(log^2(n))$==

## 8.8 Summary
![[Screenshot 2024-04-01 at 3.00.18 PM.png]]