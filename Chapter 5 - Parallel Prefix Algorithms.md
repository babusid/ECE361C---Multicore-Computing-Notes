## 5.1 Introduction
- This chapter is about the *parallel-prefix* operation
	- Also known as *scan*
- Recall reduce operation from [[Chapter 4 - Parallel Reduce Algorithms]]
	- This is an *associative* operation, applied to an array of size *n* to get a global answer for the entire array 
		- ie. sum of entire array, min of entire array etc.
- **Parallel - Prefix** is computing these types of associative operations for each successive sub-array (prefixes).
- **Section 5.2** gives an algorithm for *parallel-prefix* that runs in *O(nlogn)*
- **Section 5.3 presents work-optimal algorithms for parallel-prefix**
	- First a simple recursive parallel algorithm
	- Then an iterative version also known as a **Blelloch scan.**
- **Section 5.4 gives an LLP version**
- **Section 5.5 presents applications of parallel-prefix**
## 5.2 $O(nlog(n))$ parallel prefix algorithm
- Suppose you have an array $A$ of size $n$. 
- We want to compute $C$, the sum of all possible prefixes of $A$.
	- **Formally defined as $C[k] = \sum_{i=0}^{i\leq k} A[i]$**
- Sequential algorithm is very obvious
	- ![[Screenshot 2024-03-31 at 11.30.05 AM.png]]
	- Runs in $O(n)$ time
	- Takes $O(n)$ work
### Par Prefix Algorithm 
- **There is a parallel algorithm that reduces the time by using the concept of a binary tree.**
	- Similar to the idea behind [[Chapter 4 - Parallel Reduce Algorithms#ParReduce Algorithm| ParReduce Algorithm]]
	- This algorithm is called **ParPrefix**
	- ![[Screenshot 2024-03-31 at 11.38.03 AM.png]]
	- Sort of a *reach-back* aggregation 
		- First time, $C[i]$ will grab $C[i]$ and $C[i-1]$ = the initial value plus the one right before it
		- Second time, $C[i]$ will *reach back* and grab $C[i-2]$,  and sum it with the current value
		- Third time, $C[i]$ will reach back again, and grab $C[i-4]$, skipping $i-3$ because the $i-2$ value in C would have included it in the first iteration 
		- so on and so forth.... 
- ParPrefix only requires $O(log(n))$ but is not work-optimal. It requires $O(nlogn)$ work, whereas the sequential algorithm only needs $O(n)$ work. 

## 5.3 Work optimal algorithms for parallel prefix
- This algorithm computes parallel prefix with ***O(n)*** work. We give two version of the same algorithm: one recursive and one iterative
- ==We will use 1-indexed arrays throughout, and assume that input arrays are of size *n* that is a power of 2, similar to previous chapters==
- ### Recursive Implementation
	- Base case is when the array size is smallest ($n == 1$)
	- Otherwise, copy input array $A$ into an input array $B$ where $B$ is half the size of A, and we copy by the following formula: $B[i] = A[2i] + A[2i-1]$. Then recurse on $B$.
	- Once recursion returns, parse returned array to find prefix at this level. 
		- To do this, recognize that even indices of the output array will correspond directly to elements in the returned array (due to power of 2 branching in the tree structure (splitting by two on each way down)) 
		- Odd indices will correspond to most prior pair plus the element from the input array that it shares an index with
		- This can be seen in the table below:
		  $C[2] = D[1], C[3] = D[1] + A[3], C[4] = D[2], C[5]=D[2]+A[5],...$
	- ![[Screenshot 2024-03-31 at 12.09.49 PM.png]]
	- ![[Screenshot 2024-03-31 at 12.12.41 PM.png]]
	- In each parallel time step, the input size is reduced by a factor of 2. Hence, we know the algorithm takes $O(log(n))$ time. 
	- Total work required is given by the recurrence relation:
		- $W(n) = W(n/2) + O(n)$ : non-recursive part is $O(n)$ at each level
		- $W(1) = O(1)$
		- Therefore, $W(n) = O(n)$
- ### Blelloch Scan (Iterative Implementation)
	- This is a non-recursive version that computes the ***exclusive*** scan, meaning that the scan array at index i contains the scan of all values before i but not i itself in the input array.
		- Going from exclusive to inclusive is very simple and can be done in one parallel step by simply doing a "vector addition" of the scan array to the input array in parallel 
	- This algorithm can be viewed like a tree algorithm that requires one ***upward*** sweep and one ***downward*** sweep.
	- #### Upward Sweep
		- let $v$ be some node index in the binary tree interpretation of the input array
		- let $L[v], R[v]$ represent the left and right child of $v$ respectively
		- The upward sweep then is defined as follows: $sum[v] = sum[L[v]]+sum[R[v]]$
		- Upward sweep is building a summation tree where each node contains the sum of the values beneath it.
	- #### Downward Sweep
		- After performing the upward sweep, you now replace the root with a 0, and start a downward sweep. 
		- The downward sweep is defined as follows:
			- $scan[L[v]] = scan[v]$
			- $scan[R[v]] = sum[L[v]] + scan[v]$
		- Downward sweep is making it so that each node contains the sum of the leaf nodes that precede it in a pre-order traversal of the tree (left-to-right)
			- Start from the root. The left child would inherit the 0 from the root's scan value that we set. The right child would take that 0 and add the *sum* value of the left child from the upward sweep. Now the left child has a scan value of 0, the root still has 0, and the right has $sum[L[root]]$. This propagates down, until the leftmost leaf node contains 0, and as you go right across the leaf nodes, they contain the values of everything that is to their left.
	- #### Tree Visualization
		![[Screenshot 2024-03-31 at 12.41.10 PM.png]]
	- #### Algorithm Pseudocode
		![[Screenshot 2024-03-31 at 12.38.59 PM.png]]
		- #officehoursquestion *the upward scan is just a parallel reduce like we discussed in [[Chapter 4 - Parallel Reduce Algorithms]]. Could we use the LLP algorithm for parallel reduce, and would it help at all? Additionally, the reduction operation here seems faster than ParReduce in the previous chapter?*
	- #### Proof of Correctness
		-  Refer to PDF, but after a complete down-sweep each vertex of the tree contains the sum of all the ***leaf*** values that precede it.
			- The values that propagate to the right at each layer are set to the sum of the left child. This sum is eventually broken out to each individual leaf on the left side of the tree.
			- This property is equal to saying that each right child node in the scan array contains the value of the left child node at it's level in the sum array
## 5.4 LLP Parallel Prefix Algorithm
- **Introduction**
	- Maintain assumptions from prior sections
		- Input array $A$ of size $n$ that is a power of 2.
	- This algorithm borrows ideas from Blelloch scan, the LLP implementation will use extra space but has less synchronization overhead.
- **Overview**
	- When $n$ is a power of 2, the summation tree of the reduce operation has $n-1$ nodes.
	- We're going to replace **the upward scan** in Blelloch scan with our own **LLP-Reduce** from [[Chapter 4 - Parallel Reduce Algorithms#4.4 LLP-Reduce Algorithm | the previous section.]] We will use this to create an extra array $S$ of the same size as the input array $A$ that contains the reduce values.
	- **We need another LLP algorithm to simulate the downward scan in the Blelloch Scan.** 
- **Downward Scan using LLP**
	- $G$ array of size $2n-1$
	- First $n-1$ elements compute scan values for the intermediate nodes inside the trees.
	- **Last $n$ elements give us the required result values for an exclusive scan.**
	- Similar to the LLP-Reduce algorithm, we use perfect binary tree indexing
		- Any node with index $i$ has its left child given by $2i$, right child given by $2i+1$, and parent given by $round-down(i/2)$  
	- **Algorithm initializes every element of $G$ to be $-\infty$**
	- Each element is updated once when the ***forbidden state*** is true for its index.
	- **What is the predicate and the forbidden state for this problem?**
		- **Predicate:**
			- We want the last n elements to represent the results of the exclusive scan. Equivalently, we want the leaf nodes of the binary tree to represent the results of the exclusive scan.
			- To accomplish this, we borrow from the Blelloch scan, and say that all nodes should have a "scan" value equal to the sum of all leaf nodes that precede it in a pre-order traversal.
			- Note: $S$ array is from the upward sweep section (simulated by LLP-Reduce)
			- $$ \begin{matrix} 
				  B(G) = \\
				  (G[1] = 0) \land
				  (\forall j: ( mod(j,2)=0), G[j] = G[j/2]) \land  \\ % left child anywhere
				  (\forall j: (1\leq j \leq n \land mod(j,2) \neq 0), G[j] = G[j/2] + S[j-1]) \land \\ % right child intermediate
				  (\forall j: (n < j \leq 2n-1), G[j] = G[j/2] + A[j-n])\\ %last layer right child
			  \end{matrix} $$
			  - Why is this the predicate? 
				  1. $G[1]$ has to be 0, as there are no elements before it. 
				  2. Even-index elements are guaranteed to be the left children of their parents. Therefore, they just get assigned their parent's G value. (Same idea as blelloch)
				  3. Odd-index elements have two cases. they are either intermediate nodes or they are leaf nodes
					  1. For case 1, we need to use their parent's G value, plus the sum value of their left side sibling from the reduce. Same idea as blelloch.
					  2. For case 2, we need to use their parent's G value, plus their value in the input array itself. We use $A[j-n]$ because the leaf nodes will start at $G[n+1]$, and that first node corresponds to $A[1]$. $(n+1) - n == 1$ which is what we need. This extends to the rest of the leaf indices.
		- **Forbidden State:**
			- To get an appropriate forbidden state, we can just invert the predicate. Equivalently, we can just interpret it is a set of "ensure" statements. 
- ### Algorithm Pseudocode
	- ![[Screenshot 2024-03-31 at 2.13.00 PM.png]]
 
## 5.5 Applications of Parallel-Prefix


## 5.6 Summary

![[Screenshot 2024-03-31 at 2.41.32 PM.png]]
## 5.7 Problems
![[Screenshot 2024-03-31 at 2.41.24 PM.png]]