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
- 
# 2.3 Notation


# 2.4 Properties of the LLP Algorithm