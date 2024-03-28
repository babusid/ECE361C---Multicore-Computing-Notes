
[[lattice (1).pdf | Reference Handout ]]


## Overview
- Lattices are a fundamental construct for[[Chapter 1 | generalized parallel algorithms]]
	- Core portion of [[Chapter 2 - Lattice Linear Predicate Detection | LLP algorithm]]
- Lattices are a graphical / mathematical construct designed to represent *[[Reference - Relations Notes|relations]]* and *sets*
- Understanding / Modeling Parallel systems **requires** an understanding of partially ordered sets. Partially ordered sets are critical to understanding lattices.
- **A partial order is a relation that satisfies certain properties**.
## A.1 Introduction / What is a relation?

- A relation over a given set *X* is simply some subset of *X \* X*
- Basically, a relationship between pairs of elements in the original set defines the relation.
- Relations can be visualized as a graph - nodes are elements of the original set, pairs exist in the relation if there is an edge between two elements.
#### Properties of relations:
##### Reflexive Relations
- **Reflexive Relations** are relations where every element is related to itself under that relation.
	- Formally, for each x $\epsilon$ X, (x,x) $\in$ R if R is a reflexive relation
##### Irreflexive Relations
- **Irreflexive Relations** are the opposite of Reflexive -> no x in X is related to itself
	- Formally, for each x $\epsilon$ X, (x,x) $\not \in$ R if R is a irreflexive relation
##### Symmetric Relations
- **Symmetric Relations** are relations where the existence of a pair in the relation implies that the flipped pair is also in the relation
	- Formally, (x,y) $\in$ R implies that (y,x) $\in$ R if R is a symmetric relation
##### Antisymmetric Relations
- **Antisymmetric Relations** are relations where if a pair and its inverse both exist in the relation, it implies that both elements of the pair are equivalent
	- Formally, if (x,y) $\in$ R and (y,x) $\in$ R, then y = x, given that R is an antisymmetric relation
##### Transitive Relations
- **Transitive Relations** are relations where if one element is related to a second element, which is related to a third, then the first and third are also related
	- Formally, if (x,y) $\in$ R and (y,z) $\in$ R, then (x,z) $\in$ R, given that R is an transitive relation
##### Equivalence Relations
- ***Equivalence Relations*** are [[#Reflexive Relations|reflexive]], [[#Symmetric Relations|symmetric]], and [[#Transitive Relations|transitive]]
	- **Must be all three**
#### Equivalence Relations and Equivalence Classes
- Under an equivalence relation R, for every element *x* $\in$ X, there is a set of elements *y* that comprise the ***equivalence class** of x*, defined as **all** elements where *(x,y) $\in$ R*
- The set of **all** equivalence classes under an **equivalence relation** forms a ***partition*** of X
## What is a Partial Order? (A.2 Definition of Partial Orders)
### Reflexive Partial Order
- A relation R is a ***reflexive partial order*** if it is **[[#Reflexive Relations|reflexive]], [[#Antisymmetric Relations|antisymmetric]], and [[#Transitive Relations|transitive]].**
	- Divides relation on the set of natural numbers
- Denoted by (X, $\leq$)
### Irreflexive Partial order
- A relation R is a ***irreflexive partial order*** if it is [[#Irreflexive Relations|irreflexive]] and [[#Transitive Relations|transitive]]
	- **Note that there is no requirement for symmetry or antisymmetry** 
	- Less than relation on the set of natural numbers 
- Denoted by (X, $<$)
- **For purposes of this course, irreflexive partial order will be shortened to *poset***

### Total Order
- A relation is a total order if R is a total order and for every possible pair of elements, one of their (two possible) pairings exists in the relation
	- Formally, $\forall$ x,y $\in$ X, (x,y) $\in$ R OR (y,x) $\in$ R 
- **Basically, every two elements is in some way related or comparable**

### Finite Posets
- Remember, [[#Irreflexive Partial order|poset]] means an irreflexive partial order: 
	- no element related to itself
	- transitive relation
- FInite posets are often graphically depicted using a **Hasse diagram**
- Hasse diagrams are a built on top of the **cover** relation
- #### What is Cover?
	- For any pair of elements, x covers y if x is less than y and there is some other element z such that if x is less than or equal to z and z is less than y, z must equal x. Equivalently, if x is less than y, there cannot be any element such that x is less than z and z is less than y.
	- Effectively, it means you cannot have an element (x) be related to another element (y) when y is related to an element (z) that x is also related to directly. x must be related to z through y.   
- #### Hasse Diagram
	- Hasse diagram of a poset is a graph where there is an edge between two elements x and y if and only if **y covers x**
- **Poset Elements are comparable** if you can say $x < y$ or $y < x$. 
- **Poset Elements are incomparable** if you cannot say $x < y$ or $y < x$, and is denoted as x||y.
- A poset is called a ***chain*** if every pair of elements is comparable
- A poset is called an ***antichain*** if every pair of elements is incomparable
- A **chain** is called a ***maximum chain*** if no other chain contains more points than it
- **Poset Height** is number of points in the max chain
- **Poset Width** is the number of points in max antichain

## A.3 Lattices

- Meet and join are operators defined on ***subsets*** of a poset X. They are also how we *define* lattices.
	- Join is also known as supremeum (sup)
	- Meet is also known as Infimum (inf)
#### Definition of Join
- Let $Y \subset X$ where (X, $\leq$) is a poset. 
	- **For any $m \in X$ we say m sup Y or m is the join of Y if it satisfies the following:**
		- **$\forall y \in Y: m\geq y$**
			- This condition says that *m* is the upper bound of the set *Y*.
		- **$\forall m' \in X: (\forall y \in Y: m' \geq y) \implies m' \geq m$**
			- This condition says that if *m'* is another upper bound of set *Y* then it is *greater than m.*  
	- Join is also known as the ***lowest upper bound.***
#### Definition of Meet	-  Let $Y \subset X$ where (X, $\leq$) is a poset. 
- **For any $m \in X$ we say m inf Y or m is the meet of Y if it satisfies the following:**
		- **$\forall y \in Y: m\leq y$**
			- This condition says that *m* is the lower bound of the set *Y.*
		- **$\forall m' \in X: (\forall y \in Y: m' \leq y) \implies m' \leq m$**
			- This condition says that if *m'* is another lower bound of set *Y* then it is *less than m.*
	- Meet is also known as the ***greatest lower bound.***
	
- **A poset is a lattice if and only if for every pair of elements in the poset, both the meet and join of x and y exist.**
#### Distributive Lattices
- $\forall x,y,z \in X: x \cap (y \cup z) = (x\cap y)\cup(x\cap z)$
	- Above property is equivalent to:
	- $\forall x,y,z \in X: x\cup(y\cap z) = (x\cup y)\cap(x\cup z)$
	- **In a distributive lattice, $\cup$ and $\cap$ operators distribute over each other.**
	- **Any power-set lattice is distributive.**
	- Divides relation is also distributive

## A.4 Properties of Functions on Posets
- Let (X, $\leq$) and (Y, $\leq$) be two separate posets
- A function $f: X \to Y$ is called *monotone* if and only if:
	- $\forall x_1,x_2 \in X: x_1 \leq x_2 \implies f(x_1) \leq f(x_2)$
- Effectively, monotone means ordering is maintained during the mapping between posets.

## A.5 Down-Sets and Up-Set
- Let (X,$\leq$) be some poset. 
- We call a subset Y$\subset$ X a down-set if:
	- $z \in Y \cap y < z \implies y \in Y$
- We call a subset Y$\subset$ X a up-set if:
	- $y \in Y \cap y < z \implies z \in Y$

- Down sets are more useful than up-sets for distributed systems.