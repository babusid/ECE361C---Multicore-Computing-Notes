[[bk-chapter2.pdf|Reference PDF]]
# 2.1 Introduction
- LLP algorithm can be used to solve a wide variety of very generic problems
- LLP algorithm is built around the concept of finding an element in a special lattice that satisfies a predicate (specifically the least element) and converting other problems to this setting. 
- LLP Algorithm is naturally *nondeterministic, parallel, and online.*

# 2.2 Lattice - Linear Predicates
- Let **L** be the lattice of all ***n-dimensional*** vectors of real numbers **greater than or equal to** the zero vector, **and** **less than or equal to** a given vector *T* where the order on the vectors is defined by the component-wise natural.
	- Note that definition of [[Reference - Lattice Notes#A.3 Lattices|lattice]] is that it is a [[Reference - Lattice Notes#Finite Posets|poset]] with meet/join defined for every element pair
	- More concretely, L is a lattice of vectors where
		- $L: \forall k \in R^n,k \geq \mathbf{0}, k\leq \mathbf{T}$
			- less than and greater than are defined by component wise natural comparison
		- Minimum element of L is the **zero vector.**
- We use this lattice to model the search space of the **combinatorial optimization problem.**
	- Note: this lattice is comprised of vectors of non-negative reals - but results are applicable to any distributive lattice
- **Combinatorial Optimization Problem == finding minimum element in the lattice that satisfies some predicate that models feasible / acceptable solutions**
	- We are wanting parallel algorithms that can solve the optimization problem with *n* processes.
	- We will assume that the system maintains a Global State Vector **G** $\in$ L (lattice) that contains the current solution "candidates". Each element $G[i]$ is maintained at process i
		- G == global state
		- $G[i]$ == state of process i
- Combinatorial Optimization is a special case of **predicate detection problem.** PDP == finding *some* solution that satisfies a predicate modeling acceptable solutions. CO == finding *minimum* solution that satisfies that predicate
- ==**Lattice-Linearity is a property of predicates which enables the efficient computation of minimum elements.***==
	- Ie. we want predicates to be lattice-linear so that solving the CO problem is feasible
## Definition of Lattice - Linearity
- Before we define Lattice-Linearity, we need to define ***forbidden states.*** Forbidden states are needed for efficient predicate detection within a search space.
-  ==**Definition of Forbidden States==: When given a predicate B, and a state vector G which is an element of the search space lattice L, forbidden states are elements of G if there are other state vectors greater than or equal to G which fail the predicate BECAUSE they contain that same element.**
	- Formally:
	  Given any distributive lattice $L$ of $n$-dimensional vectors and a predicate $B$, we define $forbidden(G,i,B)$ = $\forall H \in L : G \leq H : (G[i] = H[i]) \implies \not B(H)$
- 
- ==**Definition of Lattice-Linearity**: **We say a predicate $B$ is lattice-linear with respect to a lattice *L*  if for any Global State *G*, *B* is false in *G* implies that *G* contains a *forbidden state.***==
	- Formally:
		  A boolean predicate *B* is lattice-linear with respect to a lattice *L* if AND ONLY IF $\forall G \in L: \not B(G) \implies \exists i: forbidden(G,i,B)$
## ==LLP Algorithm==
#llpalgo
- Note: This is probably the most important part of this chapter
- Setup: Given any lattice-linear predicate *B* suppose $\not B(G)$ . This means that *G* must be **advanced*** on all indices *j* such that *forbidden(G,j,B).* 
- We use some function $\alpha(G,j,B)$ such that modifies the Global State ***G*** in order to **advance*** it. By advancing, it means modifying it such that we get closer to a possible solution
	- The point of the LLP algorithm is to take in a predicate we are trying to optimize for, and an input / initial state vector. The alpha functions job is to (element-wise) modify that global state vector (which is an element of our search lattice) such that the global state is less than or equal to the input vector, and becomes not forbidden.
- ![[assets/Screenshot 2024-03-29 at 11.16.05 AM.png]]
- **TLDR: LLP algorithm requires defining a forbidden state for the predicate as well as a way to advance / get closer to a correct solution. Then, it can work towards a solution in parallel easily.**
# 2.3 Notation


# 2.4 Properties of the LLP Algorithm