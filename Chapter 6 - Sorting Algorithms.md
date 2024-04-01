## 6.1 Introduction
- Ordering / Sorting elements is a fundamental problem in computing
- In this chapter we will focus on designing algorithms based on the notion of *ordering* a set of elements of size *n*.
	- We will assume that order is total, ie. every two elements is comparable
- Before we go into sorting algorithms, lets first look at the search problem
### Searching for an element in a sorted array
- This is very easy sequentially in $O(n)$
- Since the array is sorted, we can also use binary search to do it in $O(log(n))$
- Binary search is a divide-and-conquer strategy which easily maps to multiple parallel processors

### Chapter layout
- **Section 6.2:** sorting algorithms based on swapping consecutive entries that are out of order. This is a general class of algorithms that are very natural. However, they take $O(n^2)$ sequential time and $O(n)$ parallel time in worst case.
- **Section 6.3:** Sequential AND parallel version of *mergesort* 
- **Section 6.4:** Sequential AND parallel versions of *quicksort*.
- **Section 6.5: Bitonic sort**
- **Section 6.6: A sorting algorithm that is *not* based on the comparison of two entries.**

## 6.2 - General Class of algorithms
- Let $A$ be an array of distinct integers. Our goal is to sort the array.
- Mapping this problem to a lattice-search problem is not immediately apparent. However, if you view the problem as *finding the appropriate permutation $\pi$ such that applying it to the input array results in a sorted array,* you now have a search space that you can optimize.
	- For example, if $A = [45,12,15]$, then $\pi = [3,1,2]$ will sort the array A into $S = [12,15,45]$ (values in $\pi$ signifies where the element at that index in A should move to in S $\equiv$ entry $i$ in A goes to $\pi(i)$)
- Many ways to represent a permutation. 
- Let the ***inversion number*** of any entry $i$ as the number of entries less than $i$ that appear to the right of $i$ in the permutation. 
	- Permutation $[3,1,2]$ can be written using inversions as $[2,0,0]$ because there are two numbers less than 3 that appear to the right of 3, 0 numbers less than 1 to the right of 1, and 0 numbers less than 2 to the right of 2.
- **Inversions and Permutations have a 1-1 mapping.**
- Total number of possible inversion vectors are $n!$
	- last entry must always be 0 (nothing to its right)
	- first entry can be $0...n-1$ 
	- second entry can be $0...n-2$ 
	- so on
- **We define an order between inversion vectors based on component-wise vector ordering**
	- The set of all inversions forms a finite distributive lattice under this order because it is just a set of real-valued vectors
- Bottom element in this lattice is the 0 vector, which corresponds to the identity permutation
	- Identity permutation is $[1,2,3,4,...,n]$ which means nothing moves in A. This would map to a 0 vector inversion because the permutation vector is an ordered list.
- **Goal is to find an inversion vector $G$ such that applying it will give us the sorted array.**
- #### LLP-Sort 1
	- $G$ is initialized to the zero inversion vector (nothing moves when you apply this.)
	- **We need to find a *feasible* inversion vector that will sort the array.**
	- Suppose we have $G$ such that applying it, the array $A$ will not be sorted.
		- This means there exists an index $i$ that still violates sortedness $\equiv$ there exists $j>1, A[j] < A[i]$ 
	- Any inversion vector in which the number of inversions (inversion operations) for an index $i$ does not increase compared to the failed vector $G$ cannot result in a sorted list $A$
	- To check for forbidden indices, instead of checking for all violations, we only check for ***immediate* violations** in our first algorithm. 
		- ie. If $A[i] > A[i+1]$ upon applying $G$, it is clear that $i$ is forbidden in G with respect to the definition of sortedness. Swapping $A[i]$ and $A[i+1]$ will advance the $G$ vector by incrementing $G[i]$.
			- Incrementing $G[i]$ is equivalent to modifying the permutation vector such that one element has to move 1 spot further $\equiv$ a swap
		- ***Checking for only immediate violations is fine, because even if you have long-range violations due to chunks being misplaced, there will always be a violation at the chunk border.***
			- ie. $[2,3,4,1,5]$, although there is no immediate violation between 2,1 there is between 4,1. This will be fixed, and the fix will over multiple cycles propagate until you get the right order.
	- **Instead of first finding the *correct* inversion vector and then applying it, in this algorithm we iteratively apply various inversions as we travel the inversion lattice searching for the optimal inversion vector.**
		- We don't even maintain the inversion vector; we just maintain the effect of applying it.
	- ![[Screenshot 2024-03-31 at 3.31.12 PM.png]]
	- It is very important to note that **LLP-Sort1 is a non-deterministic algorithm because multiple $j$ may be forbidden at any point.** 
	- Bubble and insertion sort are both actually instances of deterministic algorithms that simply enforce an order on evaluating forbidden indices.
		- Bubble sort checks forbidden indices from 1 to $n-1$ in round robin order until no forbidden index is found
		- Insertion sort operates on the principle of maintaining a sorted array. 
			- Suppose that $A[1 ... i-1]$ is sorted. This means that the inversion vector to make this subarray sorted is the zero vector. Since there is only one entry added at the end, the inversion number for any entry / amount of violations can increase at most by one. You just have to check for violation with respect to the new entry.
- ### Odd - Even sort
	- This is an improvement on LLP-Sort1 optimized for parallel execution
	- Why do we need to optimize? **LLP-Sort1 required atomicity in checking forbidden and executing swaps. This is expensive.** 
		- This could happen if you have two forbidden indices right next to one another.
	- We can avoid this expense by scheduling forbidden indices such that conflicts / collisions are not a problem.
	- **Solution:** Schedule odd and even indices to be evaluated in two separate phases. In the first phase, only odd indices check for the forbidden state and execute the advance. In the second phase, only the even indices do. These phases alternate until there is no forbidden index left.
	- This is similar to bubble sort
	- Time complexity is $O(n)$ and takes $O(n^2)$ work (same work as bubble sort, just parallelized to be faster)

## 6.3 - MergeSort
- Merge sort is a very intuitive divide and conquer style algorithm.
- Merge Sort is traditionally implemented recursively.
- The actual sorting part is not hard, you just split and recurse until you have 1 element, and then return up. The hard part is **merging** your two sorted subarrays.
- Merging has a relatively simple sequential $O(n)$ algorithm
	- Keep two pointers for the two arrays (call them $B,C$) to merge them into a result array $D.$
	- At any step / iteration, compare $B[i]$ and $C[j]$
	- Whichever one is smaller, insert it into the next available slot in D, and increment the pointer of the array it came from (ie. if $B[i] > C[j] \implies D[next] = C[j], j=j+1$ )

- However, we would like to do this in parallel, perhaps a bit faster. *For simplicity, we are going to assume all array elements are unique.*
- This parallel approach is based on finding the location in the result array $D$ where each element of the sub-arrays $B$ and $C$ will appear. If every element in both $B$ and $C$ determines its rank (location) in parallel, it can write that value at the correct location in $D$
- We define **rank** of an element as the number of elements in the two sub-arrays that are less than the element being ranked.
	- $rank(B[i],D) = COUNT(\forall x \in B: x<B[i]) + rank(B[i],C)$
- Since the first sub-array is sorted, the first number (Count..) is ***always*** just $i$.
- Since the second sub-array is sorted, the amount less than $B[i]$ can be determined by binary searching for it in the other sub-array in $O(log(n))$ time.
- **Therefore, the overall time complexity of determining rank and therefore merging is $T(n) = O(1) + O(log(n))$**
- Therefore, parallel merge sort can be done in $O(log(log(n)))$ time
## 6.4 - QuickSort
- Quicksort is very fast
- $O(n^2)$ worst case time complexity
- $O(nlog(n))$ average case time complexity
- **Algorithm is based on partitioning the array into two parts based on a pivot**
- Once the lower half is less than or equal to the pivot and the upper half is greater than the pivot, you can recurse on each half. **Since sorting on each half is independent, you can sort the subarrays in parallel.**
	- Note: the sub-arrays do not need to be in sorted order before you recurse. You just need to treat them as "smaller than" and "bigger than" bags relative to the pivot.
- There are many methods to chose a pivot to partition. Random choice is easy, and will on average result in approximately equal sized partitions. This gives us the average case ***sequential time complexity*** of $O(nlog(n))$. Worst case, the recursion only reduces the range by only 1, resulting in the worst case ***sequential time complexity*** of $O(n^2)$
- **We now describe a variant of the Quicksort algorithm in which the array is partitioned into *three* parts.**
- ### Three - way Partition Algorithm
- This algorithm takes in: Input array $A$, *low* and *high* as input parameters. The algorithm sorts the region $(A[i] \vert low \leq i < high)$
	- Note that when low and high are the same, the range is empty and the algorithm does nothing.
- We define a function called $partition$ that splits the range into three parts.
	- Low partition: All elements in the range $[l_o ... p)$ are strictly less than the pivot
	- Mid partition: All elements in the range $[p ... q)$ are *equal* to the pivot
	- High partition: All elements in the range $[q ... high]$ are greater than the pivot.
- We only need to recurse on the low and high partitions, depending on the pivot, either (or both) of the first and third partitions could be empty.
- **How does $partition$ actually work?**![[Screenshot 2024-03-31 at 5.41.12 PM.png]]
	- $while$ loop maintains the conditions for the three ranges.
	- Initializes $p,q$ to $low$ making the first two partitions initially empty to trivially satisfy the conditions. Also initializes $k$ to $high$ to make the top range empty, and ensuring that condition is also trivially satisfied
	- The range $[q ... k)$ is the initial input, and initially is a mixed bag relative to the pivot
	- In each iteration, the while loop reduces this range by either increasing $q$ or decreasing $k$
- Parallel quicksort in ch12
## 6.5 - Bitonic Sort
### Context
- **Bitonic sort is a natively parallel sorting algorithm.**
- **Bitonic sort is an *oblivious* sorting algorithm, an extremely nice property**
- #### Oblivious sorting
- *Oblivious* (for a sorting algorithm) means that the control flow of the algorithm is independent of the actual data that is being operated on.
	- ie. Even if the input array is already sorted, the algorithm runs the exact same way. This is a nice property because now the execution is highly predictable and deterministic, making it nice to work with from a hardware perspective or from a GPU development perspective.
- Oblivious sorting can be built with comparators - blocks that take in two inputs, and output $lo, hi$ where the former is the lower input and the latter is the larger.
- By combining comparators, you can create a *sorting network* that produces a sorted output from any input combination
- #### Zero - One Principle
	- Before getting into bitonic sort, first we have to understand this principle. It is **very useful for understanding sorting-network based algorithms.**
	- This principle states that **if a sorting network gives correct output on all inputs that consist of only zeroes and ones, then it will give correct output on all inputs**
		- ie. If you can show zero-one principle, analyzing sorting networks becomes simpler because you only need to consider binary inputs.
	- ##### Zero-one principle proof
		- Suppose instead of feeding comparator $(x_1, x_2)$ you fed it ($f(x_1), f(x_2)$).
		- If $f$ is a monotonic function (ie. $(x_1 \leq x_2) \implies (f(x_1) \leq f(x_2))$, it is clear $f$ distributes over $min$ and $max$.
		- Hence, when ($f(x_1),f(x_2)$) is presented to a comparator, it outputs $f(min(x_1,x_2))$ and $f(max(x_1,x_2))$
		- By applying induction of the number of comparators, if an input sequence $x$ is applied and any wire in the network has a value $x_i$, then it will have value $f(x_i)$ if $f(x)$ is applied.
		
		- Suppose a network sorts all $zero-one$ sequences correctly, but not a sequence of arbitrary numbers. Let that sequence be $(x_1,x_2, ..., x_n)$ such that the network mis-sorts $x_i, x_j$
			- ie. $location(x_i) < location(x_j), value(x_i) > value(x_j)$
		- We now show a zero-one sequence in which the network  gives wrong answers. We use a monotonic function $f$ that maps every element in x to 0 if it is less than or equal to $x_i$ or 1 otherwise. This reverse mapping would result in an incorrectly sorted zero-one sequence. But we know the network works for zero-one sequences.
- Given the proof, we know that any network that sorts zero-one sequences properly also sorts arbitrary sequences properly.
### Actual Algorithm
- Bitonic sort is somewhat similar to Mergesort in that it relies on merging two sorted subsequences. 
- **The Merge in Bitonic sort is easily parallelizable, and is called bitonic-merge.** 
#### Bitonic Sequences
- First let us define **Bitonic Sequence.**
	- A **Bitonic Sequence** is conveniently viewed as a circular array
	- We define a range $A[l ... h]$ in a circular array $A$ as follows. 
		- If $l \leq h$, then this range equals: $(A[i] \vert l \leq i \leq h)$
		- If $l > h$, then this range equals: $(A[i] \vert l \leq i \leq n-1)$
	- In other words, we travers all indices going from $l$ to $h$ **by incrementing the index by 1 mod $n$**.
		- For example, if $A = [2,4,6,8]$ then $A[2..3] = [6,8]$ but $A[3,2] = [8,2,4,6]$
			- You start at the element after l, but h is included
	- Definition of ascending sequence
		- $\forall i: 0\leq n < n-1 : A[i] \leq A[i+1]$
	- Definition of descending sequence
		- $\forall i: 0 \leq n < n-1 : A[i] \geq A[i+1]$
	- ***A sequence is bitonic if there exists two indices l and h such that $A[l .. h]$ is an ascending sequence and $A[h..l]$ is a descending sequence.***
	- It is easily verifiable that $A[l], A[h]$ correspond to minimum and maximum values in the array respectively
#### Bitonic Merge Lemma / Bitonic Merge Operation 
- Now, the **Bitonic Merge Lemma** (which enables the bitonic-merge operation)
	- Let $A$ be a bitonic sequence of length $2n$. Consider $B, C$ as follows:
		- $B = [min(A[i, A[i+n] \vert 0 \leq i < n])]$
			- each element of B is the smallest between A at i, or the element at the "other end" of the circular array.
		- $C = [max(A[i, A[i+n] \vert 0 \leq i < n])]$
			- each element of C is similarly defined to B, but with max instead
		- **Then B, C are bitonic, AND all values in B are less than or equal to all values in C.**
#### Bitonic Sort Algorithm
- Bitonic Merge operation is easily parallelizable *using a comparator network.*
- Instead of sorting both halves of the array in the same direction (a la mergesort), we sort the first half in ascending order and the second in descending order. The resulting array is a **bitonic sequence** and we can apply **bitonic merge** to it.
- **Bitonic Merge** splits the array into two parts *using the Bitonic Merge Lemma*. **Then** it does pairwise comparison to create a sorted array.
- The algorithm recursively calls itself on the left and right half arrays. Each of these calls can occur in parallel. BitonicMerge first does $n/2$ comparisons in parallel. It then recursively calls bitonicMerge on both halves. Since we halve input each time, and all comparisons are in parallel, it takes $O(log(n))$ time. 
- Parallel Time $T(n) = T(n/2) + O(log(n)), T(1) = O(1)$. Therefore, $T(n) = T(n/2) + log(n) = O(log^2(n))$
##### TLDR HOW ALGORITHM WORKS
1. Recurse down, splitting $A$ into left and right. You want to sort left in ascending, right in descending
2. When you return from a recursion, you have a bitonic subsequence
3. You use bitonic merge lemma to split that subsequence into two bitonic subsequences
	- Subsequence bitonic property is nice because you know they are sorted 
4. You do pairwise comparison and stitch the two subsequences together into a non-bitonic ascending or descending array depending on which direction you want it sorted
5. Return upwards
## 6.6 - Sorting without pairwise comparison
TODO: Later - not needed for test 2 :)

