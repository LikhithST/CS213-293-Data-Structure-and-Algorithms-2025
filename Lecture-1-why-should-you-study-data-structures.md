# Why Should You Study "Data Structures"?

## 1. What is Data?

Things themselves are not data; rather, **information about things is data**.

- **Examples of Data:** Age of people ($21$), height of trees ($14.5\text{ m}$), price of stocks ($\$182.50$), and number of likes on a post ($10\,420$).
- **The Era of Big Data:** We live in an era where data is massive. We must **process** this data to solve real-world **problems**—such as predicting the weather, finding a webpage, or recognizing a fingerprint.
- **The Cost of Disorganized Data:** Disorganized data requires a significant amount of time to process (for example, the time required to find an element in an unsorted array).

> [!NOTE]
> **Central Goal:** We can exploit structure in organized data to **design efficient algorithms** that solve our problems. This is the primary goal of this course.

---

## 2. Problem

A computational **problem** is formally defined as a pair of specifications:
1. **Input Specification:** The given inputs and their conditions.
2. **Output Specification:** The desired result that must be produced for valid inputs.

$$\text{Problem} = \langle \text{Input Specification}, \; \text{Output Specification} \rangle$$

### Example: The Problem of Search

| Component | Specification |
|---|---|
| **Input Specification** | An array $S$ of elements and an element $e$. |
| **Output Specification** | The position (index) of $e$ in $S$ if it exists. If it is not found, return $-1$. |

> [!NOTE]
> Output specifications refer directly to the variables defined in the input specifications.

### Exercise: Multiple Occurrences
**Question:** According to the specification, what should happen if $e$ occurs multiple times in $S$?

**Answer:** The output specification requires returning the position of $e$ in $S$ if it exists. Since it does not specify returning the *first* or *last* position, returning the index of **any** valid occurrence of $e$ satisfies the specification. If a particular occurrence is required, that must be explicitly stated in the output specification.

---

## 3. Algorithm

An **algorithm** solves a given problem by taking valid inputs and producing the corresponding outputs:

- $\text{Input} \in \text{Input Specifications}$
- $\text{Output} \in \text{Output Specifications}$

> [!NOTE]
> There can be many different algorithms to solve the same problem.

### Exercise 1.4: Algorithms vs. Programs

1. **What truly is an algorithm?**
   - An algorithm is a step-by-step process that processes a small amount of data in each step and eventually computes the output.
   - *Commentary:* It took the genius of Alan Turing to provide a precise mathematical definition of an algorithm (which is studied formally in CS310).

2. **How is an algorithm different from a program?**
   - An **algorithm** is an abstract, step-by-step procedure to solve a problem, independent of any specific programming language or machine.
   - A **program** is a concrete implementation of an algorithm written in a specific programming language (like C++) that can be compiled and executed on a computer.

---

## 4. Linear Search on Unstructured Data

When data in an array $S$ has no special order or structure, an algorithm must inspect elements one by one.

### Example Algorithm for Search

```cpp
int search(int* S, int n, int e) {
    // n is length of array S
    // We are looking for element e in S
    for (int i = 0; i < n; i++) {
        if (S[i] == e) {
            return i;
        }
    }
    return -1;
}
```

### Note: Pointers and Array Decay in C/C++

In C and C++, `int* S` and `int S[]` are completely identical when used in a function parameter list:
- When you pass an array to a function, the array automatically **decays into a pointer** to its first element.
- Because of this, the compiler rewrites array notation in parameter lists:
  - `int S[]` is converted by the compiler to `int* S`.
  - `int S[10]` is also converted to `int* S` (the size inside brackets is ignored).
- Both declarations mean the exact same thing to the compiler: $S$ is a pointer to the first integer in memory.

**Why `S[i]` works with a pointer:**
The subscript operator `S[i]` is syntactic sugar for pointer arithmetic and dereferencing:

$$S[i] \equiv *(S + i)$$

Since $S$ points to the start of the array, $S + i$ calculates the memory address of the $i$-th element, and `*` dereferences it to read the value from memory.

---

### Exercise 1.5: Running Time Analysis

**Question:** What is the running time of the search algorithm if $e$ is not in $S$?

When $e \notin S$, the search fails and the loop completes all $n$ iterations before returning $-1$.

![From C++ source to instruction set](Images/cpp-to-instruction-set.png)

We categorize the execution costs into four types of machine-level operations:
- $T_{\text{Read}}$: Cost of a memory access ($S[i]$).
- $T_{\text{Arith}}$: Cost of basic arithmetic, assignments, and comparisons.
- $T_{\text{Jump}}$: Cost of jump and branch instructions (conditional and unconditional).
- $T_{\text{Return}}$: Cost of function exit / return.

#### 1. Arithmetic & Comparison Operations: $(3n + 2) T_{\text{Arith}}$

- **Initialization (`i = 0`):** Runs **$1$** time.
- **Loop condition evaluation (`i < n`):** Evaluated **$n + 1$** times ($n$ times when true, plus $1$ final check when $i = n$ to exit the loop).
- **Loop increment (`i++`):** Runs **$n$** times (at the end of each iteration).
- **Equality check (`S[i] == e`):** Runs **$n$** times (checked once per iteration).

Adding these together:

$$1 + (n + 1) + n + n = 3n + 2$$

#### 2. Jump & Branch Operations: $(2n + 1) T_{\text{Jump}}$

- **Initial jump to loop condition:** In standard compiler code generation (such as line 8 `jmp .L2`), the program performs **$1$** unconditional jump to reach the initial condition check.
- **Loop condition branch:** The conditional branch to repeat the loop body (line 25 `jl .L5`) is taken **$n$** times.
- **Equality branch:** The conditional branch when the element is not found in that iteration (line 17 `jne .L3`) is taken **$n$** times.

Adding these together:

$$1 + n + n = 2n + 1$$

#### 3. Memory Accesses: $n T_{\text{Read}}$

- In each of the $n$ iterations, there is **$1$** memory read to fetch $S[i]$, giving a total of **$n$** memory accesses.

#### 4. Return Operation: $1 \cdot T_{\text{Return}}$

- **`return -1;`:** Executed exactly **$1$** time after the loop finishes.

#### Total Execution Time

Summing the individual costs produces the final analytical expression:

$$T(n) = n T_{\text{Read}} + (3n + 2) T_{\text{Arith}} + (2n + 1) T_{\text{Jump}} + T_{\text{Return}}$$

> [!TIP]
> **Godbolt Assembly Verification:** You can pass this C++ program to [Godbolt Compiler Explorer](https://godbolt.org/) and inspect the generated assembly instructions to verify that this breakdown faithfully models the generated machine code.

---

## 5. Structured Data Helps Us Solve Problems Faster

When data is organized with structure, we can exploit that structure to design much more efficient algorithms.

### Example 1.6: Search on Well-Structured Data

| Component | Specification |
|---|---|
| **Input Specification** | A **non-decreasing (sorted)** array $S$ and an element $e$. |
| **Output Specification** | Position of $e$ in $S$ if it exists. If it is not found, return $-1$. |

### Exploiting the Sorted Structure

Let us search for $68$ in a sorted array:
1. Look at the middle point of the array.
2. Since the value at the middle point is less than $68$, and the array is sorted in non-decreasing order, $68$ cannot appear in the lower half.
3. We search only in the upper half.
4. We have **halved our search space** in a single comparison.
5. We recursively repeat this process, halving the search space at each step.

![Binary search](Images/binary-search.png)

---

## 6. Binary Search

### Example 1.7: Binary Search Algorithm

```cpp
int BinarySearch(int* S, int n, int e) {
    // S is a sorted array
    int first = 0, last = n;
    int mid = (first + last) / 2;

    while (first < last) {
        if (S[mid] == e) return mid;
        if (S[mid] > e) {
            last = mid;
        } else {
            first = mid + 1;
        }
        mid = (first + last) / 2;
    }
    return -1;
}
```

---

### Exercise 1.6: Running Time when $n = 2^k - 1$ and $S[0] > e$

**Question:** Let $n = 2^k - 1$. How much time and how many operations will it take to run the algorithm if $S[0] > e$?

#### Step-by-Step Analysis

Because the array $S$ is non-decreasing and $S[0] > e$, the target $e$ is strictly smaller than the very first element, and therefore smaller than every element in $S$:

$$e < S[0] \le S[1] \le \dots \le S[n-1]$$

1. **Step 1:** The algorithm checks the middle element $S[\text{mid}]$. Since $S[\text{mid}] \ge S[0] > e$, the condition $S[\text{mid}] > e$ holds true.
2. **Subsequent Steps:** Because $e < S[0] \le S[\text{mid}]$, every iteration takes the branch `last = mid`, discarding the entire right half of the search space and continuing only on the left subarray.
3. **Total Halvings of Search Space:**
   Starting from size $n = 2^k - 1$:
   - After 1st halving: search space size is $2^{k-1} - 1$
   - After 2nd halving: search space size is $2^{k-2} - 1$
   - $\dots$
   - After $(k-1)$-th halving: search space size is $2^1 - 1 = 1$
   - After $k$-th halving: search space size is $2^0 - 1 = 0$ (interval becomes empty, loop terminates)

The search space of size $n = 2^k - 1$ is repeatedly halved until the base case is reached:

$$\log_2(n + 1) = \log_2(2^k) = k \text{ iterations / comparisons}$$

#### Comparison of Efficiency

| Algorithm | Input Structure Required | Operations for Size $n = 2^k - 1$ (Failed Search) |
|---|---|---|
| **Linear Search** | Unstructured array | Inspects all $n = 2^k - 1$ elements ($3n + 2$ arithmetic operations, $2n + 1$ jumps) |
| **Binary Search** | Sorted array | Only $k = \log_2(n + 1)$ halvings / comparisons |

For example, when $n = 1\,048\,575 = 2^{20} - 1$:
- Linear search requires over $1\,000\,000$ iterations.
- Binary search requires only $k = 20$ comparisons.
