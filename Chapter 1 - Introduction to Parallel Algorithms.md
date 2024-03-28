[[bk-chapter1 (2).pdf|Chapter 1 PDF]]
# 1.1 Introduction to LLP
- Linear programming provides insights into a lot of problems.
	- Very common paradigm.
- **Lattice Linear Predicate Detection** is another paradigm that can solve many problems.
	- This method can be used to solve **the general form** of many ***fundamental*** combinatorial optimization problems.
		- Stable marriage 
			- Classically this is solved by Gale-shapley
		- Shortest path
			- Classically this is solved by djikstra's 
		- Assigment problem
			- Classically this is solved by Kuhn's Hungarian method
- LLPD is a technique not only to solve the three problems (among others) highlighted above, but ***harder, more general*** versions of each of them. 
- LLP Models additional constraints on the problems as a *lattice-linear* predicate. Without additional constraints, approach revers to addressing the classical versions of the problems 
### Intuition behind LLP
- LLP is a generalized method to solving a large class of problems
- LLP views the underlying search space (of possible solutions) as a [[Reference - Lattice Notes#Distributive Lattices|distributive lattice.]]
	- The set of all solutions (all stable marriages, all feasible rooted trees for shortest path, set of all market clearing prices etc.) is closed under the [[Reference - Lattice Notes#Definition of Meet - Let $Y subset X$ where (X, $ leq$) is a poset.|meet]] operation of the aforementioned lattice.
		- Note: meet == greatest lower bound of a subset of a poset.
	- If the order of these solutions is appropriately defined, **finding the optimal solution is equivalent to finding the meet**
		- ***Is the appropriate order the order of optimality? How do you order solutions in that way?***
## LLP  method
- The key to the method was noted above, in that you want to find the **meet of a distributive lattice**
- However, to do that, you need to first **create** the lattice.
- The following is LLPs set of steps to solve any combinatorial optimization problem
1. **Define a lattice of vectors, *L* such that each vector is assigned a point in the search space**
	- For stable marriage, vector corresponds to the assignment of men to women
	- For shortest path, vector assigns a cost to each node (in the graph of locations)
	- For the market clearing price problem, the vector assigns a price to each item
2. **Define a boolean predicate B that models the feasibility of each vector**
	- For stable marriage, an assignment is feasible **if and only if** it is a matching and there is no unstable pair
	- For shortest path, an assignement is feasible **if and only if** there exists a rooted spanning tree at the source vertex such tath the cost of each vertex is greater tahn the cost of traversing the path in the rooted tree
	- For market clearing, a price vector is feasible **if and only if** it is a market clearing price vector.
3. Show that the *feasibility predicate* is a *lattice-linear* predicate. 
	- **Lattice Linear Property** allows for efficient search of feasible solution
	- If any point in the search space is not feasible, it allows to make progress towards the optimal feasible solution without any need for exploring multiple solutions in the lattice.
		- Additionally, this is easily parallelizable - multiple processes can progress towards the feasible solution independently

# 1.3 The Parallel Random Access Machine

- When you are working in a parallel environment, we use 4 letters to describe simultaneous memory access.
	- E = Exclusive
	- C = Concurrent
	- R = Read
	- W = Write
- Parallel computing machines are referred to as PRAM (Parallel Random Access Machine). There are four main types:
	- EREW = Exclusive Read Exclusive Write
	- CREW = Concurrent Read Exclusive Write
	- ERCW = Exclusive Read Concurrent Write
	- CRCW = Concurrent Read Concurrent Write
- CREW and CRCW are the main PRAM models that are useful

**Refer to Onenote notes for 1.4 - 16** 
- 1.4: Reduce
- 1.5 Brents scheduling principle
- 1.6 Class NC Problems 