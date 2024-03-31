## 4.1 Introduction
- This chapter illustrates some basic ideas for designing parallel algorithms by computing the **maximum of an array of size n.**
	- These ideas are applicable to other operators: sum, product, OR, AND, etc.
- Sequential algorithms can solve this class of problems in *O(n)* time and *O(n)* work. 
- This chapter focuses on ***parallel approaches to solve the same problem.***
- **For simplicity, we assume that *n* is a power of 2 for the entirety of this chapter.**
- **For simplicity, we assume that all array elements are distinct values.**

- In section 4.2, we present **ParReduce** that takes *O(log(n))* time and *O(n)* work on an EREW machine.
	- Exclusive Read Exclusive Write
- In section 4.4, we describe a technique to combine the standard sequential algorithm with a parallel algorithm to reduce the total number of processors.
- In Section 4.4, gives the LLP algorithm for the reduce operation. This algorithm doesn't have the synchronization overhead of the ParReduce algorithm
- In Section 4.5 we consider a CRCW machine and present an algorithm that takes *O(log(log(n)))* time and *O(n)* work.

## 4.2 Reduce - ParReduce
- Suppose we have an array **A** of size ***n***.
- We want to compute sum of all of the numbers in **A**.
- Sequential algorithm is just a for loop, runs in $O(n)$
- There exists a **parallel algorithm** that works in $O(log(n))$ assuming you have *n* cores.
- Intuition of this algorithm - best viewed as building a binary tree that has array elements as leaves, intermediate sums as internal nodes, total sum as root.
	- Binary tree height is O(log(n))
	- This means, if parallel algorithm can compute **level of a binary tree in parallel in $O(1)$ time, entire algorithm has time complexity of $O(log(n))$**
### ParReduce Algorithm
- Operates on the principle of partial sums taken at each level of the tree
- Array A is turned into a tree, where the child of node $i$ are nodes $2i$ and $2i-1$.
- $h$ indicates the level of the tree.
- In the first iteration, $h$ equals 1, and partial sums are stored in the first half of the array $A$.
	- Each entry in the array A represents a node. Entries contain the sum of their children nodes, represented in later half of the array. $A[i] = A[2i-1] + A[2i]$
- In the second iteration, we reduce the size of the array that stores partial sums by an extra factor of two 
	- ie, in iteration two, we store partial sums in the first fourth of the array, in iteration three the first eighth and so on.
- Eventually, after $log(n)$ iterations, total sum is stored in $A[1]$
	- Note: this assumes that array is 1-indexed. If implementing practically, I recommend leaving the 0th index blank / unused, just for ease of indexing.
- During each iteration, the sum computation of $A[i] = A[2i-1] + A[2i]$ is executed in parallel for all eligible $i$.
	- Eligible $i$ are the ones within the partial sum array bound.
	- Note: fora ll such parallel steps, assume that all processes first do all of the reads into *private registers*, and writes are only executed **after all of the reads (by all the processes) are finished.***
- ![[Screenshot 2024-03-31 at 1.35.26 AM.png]]
- Time complexity is $O(log(n))$
- Work complexity is still $O(n)$
- Needs $n$ cores
- Total cost is cores x time complexity : $O(nlog(n))$

## 4.3 Accelerated Cascading Technique
- Combining the standard sequential algorithm with the **ParReduce** algorithm can give you something even better.
- Before applying par-reduce, split the $n$-length array into $O(n/log(n))$ blocks of size $O(log(n))$
- We use $O(n/log(n))$ processors so that each block gets assigned to one processor
- Let each processor compute the maximum or sum of its block using the sequential algorithm. 
	- For each of these blocks, that is $O(log(n))$, and since it is happening in parallel, this step is $O(log(n))$.
	- Note: Total cost here is still $O(n)$ refer to pdf for explanation
- Now you have $O(n/log(n))$ numbers (1 from each of the blocks)
- Now use the binary tree algorithm, which takes $O(logn)$ time with **only** $O(n/logn)$ processors
	- **This means total cost of step two is O(n).**
- Total cost of both steps is $O(n$), Work complexity is $O(n)$, **but time complexity is $O(log(n))$**

## 4.4 LLP-Reduce Algorithm
- ParReduce has two disadvantages:
	1. Destroys original input array
	2. Requires all threads synchronize after each iteration of parallel loop
- Solving problem 1 is simple - just make copy of input array. Can be done in $O(1)$ time with number of cores equal to the size of the array.
- Solving problem 2 is what this LLP algorithm does. 
- We will retain the assumption that the array size is a power of 2.
### LLP Reduce (SUM)
- This algorithm will compute the sum of an array and take care of both of ParReduce's disadvantages
#### **Objective:** 
- Compute an array $G$ of size $n-1$ where $G[1]$ contains the sum of the entire array, $G[2]$ contains the sum of the first half, $G[3]$ contains the sum of the second half, and so on, splitting by factors of 2 each time.
	- Note: this mirrors a binary tree 
#### Array (G) specification:
- $n$ is the amount of elements in the original input array
- For $1 \leq j < n/2$ the children of $G[j]$ are the nodes $G[2j]$ and $G[2j+1]$
	- NOTE: THE CHILDREN FOR INDICES OF G BEFORE HALF ARE IN G ITSELF
- For $n/2 \leq j < n$ the children of $G[j]$ are the nodes $A[2j-n+1]$ and $A[2j-n+2]$
	- NOTE: THE CHILDREN FOR INDICES OF G PAST HALF ARE IN THE INPUT ARRAY
		- This is because indices past half refer to the local parents of leaf nodes in the binary tree. Refer to attached figure below
- ![[Screenshot 2024-03-31 at 10.28.05 AM.png]]
#### **Predicate Definition for LLP Algorithm:**
- $B(G) \equiv (\forall j: 1 \leq j < n/2 : G[j] \geq G[2j] + G[2j+1]) \land$ $(\forall j: n/2 \leq j \leq n-1 : G[j] \geq A[2j-n+1] + A[2j-n+2])$
	- What is this saying?:
		1. First statement is saying that for the first half, you have to ensure that any node is at least equal to the sum of it's two children if not more. This makes sense, as the first half represent the nodes that are within the tree, and the sum they contain **must** contain the values of their children.
		2. Second statement is saying for the second half, the nodes must contain at least the sum of the two leaf nodes that are below them. This makes sense, as these nodes represent the final layer of the tree nodes before the leaves, and the sum they contain **must** contain the values of their children
			- #officehoursquestion *Why does the second statement use at least rather than an equals? If those nodes are the last layer of the tree prior to the leaves, should the predicate not ensure that their sum is exactly equivalent tot the leaf node summation?*
- **Proof of lattice-linearity:**
	- Recall:
		- To prove lattice linearity, you have to provide a forbidden state definition such that when given a state vector $G$, it is the least element in the lattice that satisfies the predicate if it contains no forbidden states. 
		- Forbidden states are elements of $G$ if there are other state vectors greater than or equal to $G$ that fail the predicate because they contain that same forbidden state.
		- **A boolean predicate is lattice-linear IFF failing the predicate implies the existence of a forbidden state**
	- **Forbidden state definition:**
		- Given the predicate is defined as 2 conditions that have to be true for the predicate to be true, defining the forbidden state is a matter of inverting the two statements.
		- $forbidden(G,j) \equiv (G[j] < G[2j]+G[2j+1], 1 \leq j < n/2) ] \lor$ $(G[j] \leq A[2j-n+1]+A[2j-n+2], n/2 \leq j <n)$
	- Proof:
		- This proof is fairly easy. The forbidden state is the exact inverse of the predicate we are trying to detect. If the predicate fails, one of the two clauses of the forbidden state must be true. 
		- Suppose the first clause of the predicate fails. This would mean that there is an element in the first half of the G array that is not at least the sum of its two children in G. This would satisfy the first clause of the forbidden state, making $forbidden$ true.
		- Suppose the second clause of the predicate fails. This would mean there is an element in the second half of the G array that is not at least the sum of its two children in A. This would satisfy the second clause of the forbidden state, making $forbidden$ true
#### Algorithm definition
- ![[Screenshot 2024-03-31 at 10.47.39 AM.png]]
- LLP Reduce computes G in $O(log(n))$ time, because it operates on the same tree principles as Par-Reduce (ie. using cores equal to each eligible index) 
- This algorithm can be quickly modified to support any reduction operation, sum is just an illustrative example chosen for this book.
## 4.6 Summary
## 4.7 Problems

