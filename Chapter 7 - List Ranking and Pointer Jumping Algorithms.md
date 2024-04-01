## 7.1 Introduction
- In chapters 4, 5 we covered parallel algorithms on arrays via reduce and parallel-prefix
- Advantage of arrays is that any thread can access elements in constant time.
- In many applications we have ***linked lists*** instead of arrays. This means we do not have *random* access to elements.
- As such, need new approach to reduce / parpref type problems
- **In this chapter, we introduce the idea of pointer jumping that allows for significant time complexity reduction in these types of problems.**

- **Section 7.2**  describes pointer - jumping algorithm
	- Very critical for later algorithms
- **Section 7.3** gives multiple applications of the pointer jump technique
- **Section 7.4** gives the LLP Algorithm for pointer jumping.
- **Section 7.5** discusses node coloring in a circular linked list
## 7.2 Pointer Jumping Algorithm
- In this section we discuss a **Pointer Jumping algorithm for finding the distance from the tail to  any other element in the list.**
- *Context / assumptions*
	- Suppose we have a linked list.
	- Each node *i* has a field $next[i]$ indicating the next node in the list
	- If the element $i$ is the last node, then $next[i]$ returns -1.
- **Objective: Our goal is to return an array R that returns the distance of any element from the last node in the linked list.**
- **Explanation of Algorithm / Method:**
	- Algorithm shown below in [[Chapter 7 - List Ranking and Pointer Jumping Algorithms#Distance - to - last algorithm pseudocode|next section]]
	- In the Pointer-Jumping algorithm, we first copy the $next$ pointers in an array $Q$ and then use the *pointer jumping* method to reduce the length of the longest path in the link structure.
	- ==Initially $Q$ contains direct copy of next fields== $\equiv$ Q housing pointers to direct neighbor / parent
	- ==$R$ contains distances from each node $i$ to the node in its $Q[i]$ value. Since these values are initially the node they point to next, they all initialize to one except for the tail node, which initializes to 0 (the tail node does not have a next).==
	- The pointer jumping is accomplished by setting $Q[i]$ to $Q[Q[i]]$ for every node $i$
		- ie.  Instead of pointing to your immediate neighbor / parent, move to point to your skip neighbor / grandparent.
	- The algorithm assumes that all nodes execute a while loop iteration synchronously. (Algorithm shown below)
	- Algorithm has to satisfy some properties:
		1. Variable $Q[i]$ always points to a node that is reachable from node $i$, and -1 if no node is reachable from $i$
			- indices represent nodes, and $Q[i]$ represents the current "next" node for node $i$. If you pointer jump correctly, this obviously holds true, as you had to take a path to get to whatever value is in $Q[i]$
		2. Variable $R[i]$ is equal to the length of the path from $i$ to $Q[i]$ in the linked list
			- $R$ is the result array that awe are trying to find (reference the objective above).
			- As you jump between pointers, this is easy to compute as you can count links traversed.
	- Steps:
		1. In any iteration of the while loop, we consider some node $i$ such that $Q[i] \neq null \land Q[Q[i]] \neq null$ . If this condition holds, we set $Q[i] = Q[Q[i]]$
			- This maintains the first property, as you had to go through a neighbor that you know you could access to get to your new Q value.
		2. We set $R[i] = R[i] + R[Q[i]]$ 
			- This maintains the second property, because length of path from $i$ to $Q[Q[i]]$ is the sum of path lengths from $i$ to $Q[i]$ and from $Q[i]$ to $Q[Q[i]]$
		3. **When there is no node $i$ such that it has a grandparent, every node is pointing to the tail node (a reachable node that is pointing to null). Due to property 2 on R, we know that $R[i]$ contains the length of the path from $i$ to the tail node.**
	- Analysis:
		- Initialization of Q and R lists are both O(1) because all elements can be computed in parallel
		- Algorithm takes $O(log(n))$ time
			- $n$ is the size of the list
			- Although the while loop does exist within a $\forall i$ block, it is parallel. Therefore, all of them are happening simultaneously. The inner for loop's complexity is what matters. 
			- Due to the pointer jumping, at each step, each entry in Q skips a link. **Conceptualized graphically, we are "flattening the tree" each time by a factor of 2**. This leads to an $O(log(n))$ runtime, because you will only get $O(log(n))$ evaluations of the while loop itself.
		- Algorithm does $O(nlog(n))$ work, and is therefore not work-optimal
			- This is because you need *n* cores for the parallel blocks, and the second parallel block has $O(log(n))$ TC.
		- **It is possible to reduce the work, but we are not going to go over that.**
		- **Although we only used a linked list, *pointer jumping* as an idea is applicable to any rooted tree where a node points to a parent. A linked list is really just a specific case of a tree.**
### Distance - to - last algorithm pseudocode
![[Screenshot 2024-03-31 at 9.59.31 PM.png]]

## 7.3 Applications of the Pointer Jumping technique
- The algorithm that was provided in the first section was just to familiarize ourselves with and indicate the utility of of the pointer jumping idea.
- Here are a two additional examples of where it can be used:
	1. Returning the maximum value in a linked list
	2. Converting a linked list into an array
### Example 1: Returning maximum value in a linked list
- Suppose you have a linked list of numbers. We want to find the maximum. 
	- Each node $i$ has a natural number $num[i]$ stored within the node.
	- Each node maintains a $next$ pointer as well.
- ==We can repurpose the entirety of the prior algorithm, except for **one key modification!** ==
	- When modifying the $R$ array, instead of doing what we did previously:
		- $R[i] := R[i] + R[Q[i]]$) 
	- Do this instead:
		- $R[i] := max(R[i], R[Q[i]])$
- This modifies the second property of the first algorithm to where $R[i]$ is not a path length, but rather the maximum value *on the path* between $i$ and $Q[i]$.
### Example 2: Converting a linked list into an array. 
- In the original example, we computed the distance of any node to the tail. 
- Suppose you have a linked list and want it as an array instead. 
- TODO: Solve problem
## 7.4 LLP Algorithm for pointer jumping
#llpalgo 
- List ranking is a pretty common problem in computing
- In this section, we show how it can be viewed as an **LLP Algorithm**
- The problem is defined as follows:
	1. Given a tree rooted at vertex *r* as input.
	2. Each node *i* other than the root points to another node.
	3. Root node *r* points to null.
	4. **Goal is to determine the distance of every node from the root**
- Same objective as the example pointer jumping algo we first went over, except we call it "root" instead of "tail", even though the conditions are the exact same.
- **How does this express as a predicate-detection LLP algo?**
	- We view this as a search problem for a integer-valued vector *G* that satisfies some feasibility predicate. In this problem, we want G to only be feasible if it contains in some manner the distances from every node to the root.
		- *Note: integer-valued but not just a single integer. This is explained below*
	- **Construction of G:**
		- There is one element in G for every node other than the root. 
		- Each element is comprised of two numbers: dist, next.
			- dist $\equiv$ The distance from the corresponding node of an entry to the root of the list
			- next $\equiv$ The next node - similar to the the original algorithm, starts at parent, modified by jumping
		- $G[i].dist$ is initialized to 1 for everyone
		- $G[i].next$ is initialized to the parent of $i$ for all nodes $i$
	- **Predicate we want to detect:**
		- $B(G):= \forall i : G[i].next = r$
		- We want to detect the scenario where every node is pointing to the root. 
		- We will get every node to point to the root through pointer jumping
		- *Along the way, we will track the distance from each node to the root as we advance.*
		- Proof of lattice linearity for this is trivial given that the forbidden state is just any node not pointing to the root. 
	- **Algorithm:**![[Screenshot 2024-04-01 at 12.16.53 AM.png]]
## 7.5 Node coloring in a linked list
TODO: Later - not on test 2 :)
