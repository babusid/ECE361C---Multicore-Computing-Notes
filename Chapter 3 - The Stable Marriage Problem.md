## 3.1 Introduction
- In standard stable marriage, there are *n* men and *n* women each with a **totally ordered** preference list.
- **Goal:** find a matching between men and women such that there is no instability
	- Instability: There exists a pari of man and woman such that they are not married but prefer each other over their partners
- We consider stable marriage instances with *m* men numbered 1-m, and *w* women numbered 1-w. We assume number of women *w* is at least *m* ($w \geq m$). 
- *mpref* and *wpref* specify the men and women's preferences respectively
	- ***$mpref[i][k] = j$*  if and only if woman *j* is *k'th* preference for man *i***
- In this chapter we cover first the classical solution for stable marriage (Gale Shapley) in section 3.3
- In Section 3.4, we present an algorithm that creates a stable matching from any initial matching in $O(m^2+w)$ time such that the final stable matching has the least distance from the initial state out of all possible stable proposal vectors greater than the initial. 
	- This algorithm generalizes the GS algorithm which assumes initial proposal vector to be the top choices
- **In Section 3.5 we present a parallel *LLP* version of the GS algorithm.**
	- This section also discusses constrained stable matching problem, where there may be more constraints than just the preference lists (*note: lattice-linear constraints*)
		- ie. Forced pairs that must be in the final, or forbidden pairs that are not allowed etc.
## 3.2 Proposal Vector Lattice (Used for all following algorithms)
- (Refer to [[Reference - Lattice Notes|lattice reference]] for refresher on lattice)
### Definition of Proposal Vector
- ***Proposal Vector*** is critical to the stable marriage algorithms that we focus on in this chapter
- **Male** proposal vector **G** is of dimension **m** (the number of men).
- Structure of proposal vector is as follows:
	- $G[i] == k \implies$ **man i has proposed to his $k^{th}$ preference i.e. the woman given by** $mpref[i][k]$
	- This is equivalent to saying if $mpref[i][k] == j \land G[i]==k \implies$ man *i* proposing to woman *j* 
- ==Let $\rho(G,i)$ denote the woman $mpref[i][G[i]]$==
	- This means that ==$\rho(G,i)$ represents the woman that man i has proposed to in the current proposal vector / current state==
- If G equals the unit vector (all 1's) every man has proposed to their top choice
- If G is equal to the unit vector multiplied by the amount of women (all *w*'s) then every man has proposed to their top choice
	- Each man has proposed to the highest possible number which means the largest element of the least which means the least preferred
- **For simplicity, we assume complete lists for all G (no man prefers being alone to being paired)** 
- **Set of all proposal vectors form a ==distributive lattice under the natural less than order, where meet and join are given by component wise minimum and component wise maximum respectively.==**
	- Component-wise minimum is defined as the following
		- $\forall A,B \in R^n, CME(A,B) = (min(A_1,B_1),min(A_2,B_2), ..., min(A_N,B_N))$  
	- Component-wise maximum is defined similarly, except with max operators instead of min operators
	- This lattice has $w^m$ elements
		- This is because there each man in *m* has *w* possible values in their slot in *G*

### Properties of Proposal Vectors
- In any proposal vector *G*:
	- *There is a unique matching* such that *i* and $\rho(G,i)$ are matched in *G* if the proposal by man *i* is the best for **that woman** in *G*. 
	- A man *p* is unmatched in *G* if his proposal is not the best proposal for that woman in *G*
	- A ***woman*** *q* is unmatched in *G* if she does not receive any proposal in *G*. Otherwise, she is matched with *the best proposal for her* in *G*.
	- ==All this to say, men get matched to women if the are the best that woman receives in that iteration of the state. If a man proposes, and a different proposal is better, he loses. A woman only loses if she gets 0 proposals. Otherwise, she gets matched with the best one available==
- Proposal vector *G* is ***a man-saturating matching*** if and only if no woman receives more than one proposal in G
	- Formally: $\forall i,j : i \neq j: \rho(G,i) \neq \rho(G,j)$  
		- refer to previous subsection bullet 4 for $\rho$ 
	- If *m == w* , man-saturating matching is a perfect matching (all men and women are matched)
	- If $m < w$ man-saturating *G* is when all the men are matched but some women are unmatched
	- ==We say that a matching A is less than another matching B if the proposal vector for A is less than the proposal vector for B ==
		- Standard Vector element-wise comparison, yes some vector pairs are incomparable rather than less,greater, or equal
			- This is where meet/join become important
	- **Man-optimal marriage is the *least* stable matching in the proposal lattice, and woman-optimal marriage is the *greatest* stable matching.**
- A proposal vector *G* may have blocking pairs (1+)
	- #blockingpair
	- **blocking pair** is a pair of a man and woman (*p,q*) if and only if $\rho(G,p) \neq q$  AND *p* prefers *q* to $\rho(G,p)$ AND *q* prefers *p* to any proposal she receives in *G*
		- This works even when *q* is unmatched
- ==**A proposal vector G is a stable matching if AND ONLY if it is a man-saturating matching with NO blocking pairs.**==

## 3.3 Standard Gale-Shapley Algorithm
- *Skip if needed, not really multicore / parallel, just here for a reference algorithm*
- Men propose to women going from their most preferred to their least preferred. If a man is rejected, he moves to his next choice. Women, if un-engaged, always accept. They break off their engagement if they later receive a proposal they prefer more. When an engagement is broken off, the rejected man now goes back into the queue of un-engaged men. If a man proposes to an engaged woman, and is not more desirable than her fiance, he is rejected, and continues iterating.
- Note: as algorithm progresses, partners for men can only worsen and partners for women can only improve
- Note: once a woman is engaged, she will remain engaged from then on
- Note: If number of men $\leq$ number of women, algorithm is guaranteed to terminate.
- ![[Screenshot 2024-03-29 at 8.12.58 PM.png]]
## 3.4 Upward Traversal Algorithm
- **This algorithm is more general than Gale-Shapley in two key ways.**
	- It can support an Arbitrary Initial Proposal Vector
		- Gale-Shapley always finds the *man-optimal* stable marriage. What if we want to find a stable marriage such that the men are allowed to propose such that the proposal vector is $\geq$ some vector I. For example, if I == (3,1,2,1) then the first man cannot propose to his top two choices (otherwise that would make the G vector less than I), and the third man cannot propose to his top choice.
		- This algorithm works when I $\neq$ (1,1,1....,1), which is what Gale-Shapley needs.
	- It can support an unequal number of men and women. 
		- We consider the case when the number of women $>$ the number of men, if number of men is greater, simply switch the roles.
		- If we started with all men's top choices, GS would still return man-optimal stable match, with excess women unmatched. HOWEVER, if  you start from an *arbitrary* vector, you can end up with all women getting unique proposals but there may exist an **unmatched woman who is preferred by some man over his current match.** To tackle this, there is a simple Lemma
			- Let $numw(I)$ represent the total number of unique women that have been proposed to in all vectors that are less than or equal to $I$.
			- ==**Given any stable marriage instance with m men and an initial proposal vector I there is no stable marriage for any proposal vector G $\geq$ I whenever numw(I) exceeds m.==***
				- Proof in PDF
				- Due to this, we only consider initial starting proposal vectors *I* such that the total number of women proposed until *I* is at most *m*.
- Before defining the algorithm itself, we first define a ==**forbidden state.**==
	- Note: This is still ***not*** an llp algorithm, we're just using the same forbidden state concept
	- Note: M == set of all men, m == amount of men
	- $forbidden(G,i): \exists j \in M \land j \neq i$ such that both *i* and *j* have proposed to the ***same woman*** in *G* and that woman prefers *j* OR *(j,$\rho(G,i)$)* is a blocking pair ( #blockingpair ) in G
		- 1. Can't have a man *i* paired with a woman who received a better proposal from *j*
		- 2. Can't have a man *i* paired with a woman who likes *j* more than anyone, and *j* likes better than *j's* current partner in *G*
	- **With this definition of forbidden state we can state the following:**
		- *For any proposal vector G such that $numw(G) \leq m$, there exists a man i such that forbidden(G,i) is true IF AND ONLY IF G is not a stable marriage.*
		- Proof in PDF
- **Algorithm UpwardTraversal exploits the *forbidden(G,i)* function to find the stable marriage in the proposal vector lattice.**
	- Basic idea is that if a man is forbidden in the current proposal vector, then he must go down his list until he finds either an unmatched woman or a woman who prefers him over her current match
		- Very similar to GS
	- 1. We check until all men are not forbidden
	- 2. If we have forbidden men, we advance the first forbidden man along his preference list until he is no longer forbidden, and then repeat the 1. check. If at no point can this man exit the forbidden state, a stable match is impossible. 
	- 3. If no man is forbidden , we are guaranteed a stable marriage.
	- ![[Pasted image 20240329232828.png]]
## 3.5 LLP Algorithm for Stable Matching Problem
#llpalgo 
- Refer to [[Reference - Lattice Notes]]
- Refer to [[Chapter 2 - Lattice Linear Predicate Detection#Definition of Lattice - Linearity | Definition of Lattice Linearity]]
- Refer to [[#Definition of Proposal Vector]]
- In order to create an LLP algorithm, we need to define a **lattice-linear predicate.**
- ### Predicate:
	- **An assignment / matching G is feasible for stable marriage problem if it corresponds to a perfect matching AND has no blocking pairs**
		- Recall definition of blocking pair:  
			- **blocking pair** is a pair of a man and woman (*p,q*) if and only if $\rho(G,p) \neq q$  AND *p* prefers *q* to $\rho(G,p)$ AND *q* prefers *p* to any proposal she receives in *G*
- ### Proof of Predicate Lattice-Linearity
	1. Let $\rho(G,j) = z$ (woman that corresponds to choice $G[j]$ for man $j$)
	2.  ==$j$ is forbidden in $G$ (index not the man) if there is a man $i$ such that woman $z$ prefers man $i$ AND *ONE* of the following:==
		-  ==$i$ has also been assigned $z$ in $G$==
		-  ==$i$ prefers $z$ to the woman he has been assigned== 
			- ==Note: This means ($i,j$) are a blocking pair in $G$==
		- **Formal Definition of the Forbidden State is as follows:** $\exists i : \exists k \leq G[i] : (z == mpref[i][k]) \land (rank[z][i] < rank[z][j])$   
			- Note: mpref is first indexed by man id, then preference
			- Note: rank is a 2d matrix of the women's preferences, indexed by first the woman, then the man. 
			- Formal definition says that z is lower (better) in i's proposal list, and i is lower (better) in z's rank list than j
	3. **$G$ is not a stable marriage IF AND ONLY IF there is no index $j$ where $forbidden(G,j) == true$.**
		- If G not a perfect matching, there must be at least one woman assigned to two men, resulting in less preferred man being forbidden
		- If G is a perfect matching, but has a blocking pair, the partner of the woman in the blocking pair is forbidden.
		- If G has no blocking pairs and is a perfect matching, forbidden cannot be true for any elements
	4. **Need to show that if $forbidden(G,j)$ there is no proposal vector $H$ such that $(G\leq H) \land (G[j]==H[j]) \land B(H)$ (B(H) means H is a stable marriage). This is to prove that the forbidden state is a valid forbidden state with respect to the definition of lattice-linearity.**
		1. Consider any $H$ such that $(G \leq H)\land(G[j]==H[j])$  
		2. Since $(G[j]==H[j])$ we know that $\rho(G,j) == \rho(H,j)$
		3. Let $i: \exists k \leq G[i]: (z == mpref[i][k]) \land (rank[z][i] < rank[z][j])$ 
			- Let there be an $i$ such that there any woman up to his $k^{th}$ preference  prefers a man other than $i$ to $i$. 
		4. Since $G \leq H, G[i] \leq H[i]$ we get that $\exists k \leq H[i] : (z == mpref[i][k]) \land (rank[z][i] < rank[z][j])$
			- G is less than or equal to H because of initial assumption
			- $G[i] \leq H[i]$ because of initial assumption (vectors are only comparable when one is uniformly greater or equal, which is what the initial setup is saying)
			- Otherwise, same statement as statement 3 but for H
		5. Since we can make statement 3 for vector H, you know that vector H is also forbidden
- ### LLP algorithm
	- ![[Screenshot 2024-03-31 at 12.24.42 AM.png]]

## 3.6 Summary
- ![[Screenshot 2024-03-31 at 12.25.10 AM.png]]

## 3.7 Problems
- Show that the Gale-Shapley algorithm always finds a stable marriage.
- Give an instance of the stable marriage problem where there is only one stable marriage: the last choice for all men
- Give an instance of the stable marriage problem in which the maximally parallel LLP version of the algorithm is *n* times faster than the squential Gale-Shapley algorithm 
- Show that the set of the stable marriages is closed not only under meet, but also the join operation.
	- Equivalently, prove set of stable marriages form a sublattice of the proposal lattice.
- Give an algorithm in which all men start with their least favored choice and improve their choices in the search of the stable marriage.