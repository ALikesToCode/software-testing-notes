# Chapter 3: Unit Testing Based on Graphs

Software testing is both an art and a science. In our journey to achieve reliable, high‐quality software, we have already laid the foundation by understanding general testing principles and exploring graph models in Chapters 1 and 2. In this chapter, we turn our focus to unit testing through the lens of graph theory. We will see how code can be modeled as graphs, how various coverage criteria (both structural and data flow) can be defined on these graphs, and ultimately, how these criteria drive the design of effective test cases.

> *“When we test a unit of code by modeling it as a graph, we uncover not only the obvious paths of execution but also the hidden routes where errors may lurk. This approach transforms abstract code into a tangible structure that we can analyze, reason about, and rigorously test.”*  




## 1. Introduction

Unit testing is the process of validating individual functions or methods in isolation. Traditional unit testing often focuses on executing code with concrete input values and comparing outputs. However, as systems grow more complex, it becomes increasingly beneficial to represent code using graph-based models. Two key types of graphs are used:

- **Control Flow Graphs (CFGs):** Capture the flow of control (branching, loops, etc.) within a method.
- **Data Flow Graphs:** Augment the CFG with information about where variables are defined (defs) and where they are used (uses).

By analyzing these graphs, we can define coverage criteria that ensure every part of a unit's structure is exercised—thereby increasing the likelihood of detecting hidden errors.



## 2. Modeling Code as Graphs

### 2.1 Control Flow Graphs (CFGs)

A **Control Flow Graph (CFG)** abstracts a unit of code into nodes and edges:
- **Nodes:** Represent basic blocks—continuous sequences of statements with no internal branching.
- **Edges:** Represent the flow of control from one basic block to another.

#### Deriving a CFG

When converting code into a CFG:
- **Basic Blocks:** Group contiguous statements that execute sequentially (e.g., variable initializations, assignments, print statements).
- **Decision Points:** Each branch (if, while, for, switch) creates separate nodes for the decision and its outcomes.
- **Loops:** Special handling is needed. For example, in a while loop, the loop condition is represented by one node, and the loop body by another; a back edge represents the loop iteration.

*Example 1 – If Statement with Else:*  
Consider the code fragment:  
```java
if (x < y) {
    y = 0;
    x = x + 1;
} else {
    x = y + 1;
}
z = x + 1;
```
Its CFG would include:
- An initial node for the decision.
- One branch node for the “if” block (executing the two statements).
- Another branch node for the “else” part.
- A convergence node where the final statement `z = x + 1` is executed.

*Example 2 – Loops:*  
For a while loop such as:  
```java
x = 0;
while (x < y) {
    y = f(x, y);
    x = x + 1;
}
```
The CFG would have:
- An initial node for `x = 0`.
- A node for the while condition.
- A node for the loop body.
- A back edge from the loop body back to the condition.
- An exit edge when the condition fails.

Modern IDEs (e.g., Eclipse, IntelliJ IDEA, Visual Studio) often include plugins to generate CFGs automatically. Even if you generate them automatically, understanding their structure is crucial for effective test design.

### 2.2 Data Flow Graphs

A **Data Flow Graph** enhances the CFG by annotating nodes (and sometimes edges) with definitions (defs) and uses of variables:
- **Definition (Def):** A point in the code where a variable is assigned a value.
- **Use:** A point where that variable is accessed (e.g., as part of an expression, condition, or output).

These annotations allow us to analyze how values propagate through the code. For each variable, we can define **du-pairs** (def-use pairs) representing paths from a definition to a use. A **du-path** is a simple path (without repeating nodes) from the point where a variable is defined to where it is used, with the property that the variable is not redefined along that path.

*Example – Statistics Program:*  
Consider a method that computes statistics:
```java
public static void computeStats(int[] numbers) {
    int length = numbers.length;
    double med, var, sd, mean, sum, varsum;
    sum = 0.0;
    for (int i = 0; i < length; i++) {
        sum += numbers[i];
    }
    med = numbers[length / 2];
    mean = sum / (double)length;
    varsum = 0.0;
    for (int i = 0; i < length; i++) {
        varsum += ((numbers[i] - mean) * (numbers[i] - mean));
    }
    var = varsum / (length - 1);
    sd = Math.sqrt(var);
    System.out.println("mean:" + mean);
    System.out.println("median:" + med);
    System.out.println("variance:" + var);
    System.out.println("standard deviation:" + sd);
}
```
The CFG derived from this method includes two loops (one for summing and one for computing variance). Data flow analysis identifies:
- **Defs:** Variable assignments (e.g., `sum = 0.0`, `med = numbers[length/2]`).
- **Uses:** Variables used in expressions (e.g., `numbers[i]` in the loops, `sum` in computing mean).

By mapping these definitions and uses onto the CFG, we generate du-pairs that form the basis for data flow coverage criteria.



## 3. Structural Coverage Criteria for Unit Testing

Once code is modeled as a graph, we define various **coverage criteria** that specify which parts of the graph must be exercised by our test cases.

### 3.1 Node and Edge Coverage

- **Node Coverage:**  
  Every node in the CFG must be visited by at least one test case. This criterion is analogous to statement coverage.
  
- **Edge Coverage:**  
  Every edge in the CFG must be traversed, ensuring that every transfer of control is tested. Since nodes are part of edges (edges are paths of length 1), edge coverage subsumes node coverage.

*Example:*  
For a CFG with nodes \( \{1, 2, 3\} \) and edges \(\{(1,2), (1,3), (2,3)\}\), a test case that traverses the path \([1,2,3]\) achieves both node and edge coverage.

### 3.2 Edge-Pair and Path Coverage

- **Edge-Pair Coverage:**  
  Requires that every pair of consecutive edges (paths of length 2) is executed. This criterion tests the interactions between sequential decisions.
  
- **Complete Path Coverage:**  
  Would require testing every possible path in the CFG. In the presence of loops, this criterion is infeasible due to the infinite number of paths.
  
- **Specified Path Coverage:**  
  The tester specifies a finite set of important paths, particularly useful when loops are present.

### 3.3 Prime Path Coverage

**Prime Paths** are maximal simple paths (simple paths not properly contained in any other simple path). They are highly effective for testing loops and complex branching:
- Prime path coverage ensures that all significant execution sequences are tested.
- This criterion naturally subsumes node and edge coverage but may not always cover every edge-pair.

### 3.4 Round Trip Coverage

For graphs that contain cycles:
- **Simple Round Trip Coverage:**  
  At least one round-trip (a prime path that starts and ends at the same node) must be executed for each node that can form a cycle.
  
- **Complete Round Trip Coverage:**  
  Every distinct round-trip path must be executed.

These criteria are especially important in systems where a unit of code may re-enter a particular state.



## 4. Data Flow Coverage Criteria

Data flow coverage focuses on the propagation of variable values through the code.

### 4.1 Definitions and Uses

- **Definition (Def):**  
  A location where a variable is assigned a value (e.g., through an assignment or initialization).
  
- **Use:**  
  A location where a variable’s value is accessed (e.g., in an expression, conditional, or output).

### 4.2 DU-Pairs and DU-Paths

For each variable \( v \):
- A **du-pair** is a pair \( (l_i, l_j) \) such that \( v \) is defined at \( l_i \) and used at \( l_j \) with no redefinition in between.
- A **du-path** is a simple path (no repeated nodes) from the definition to the use that is **def-clear** (i.e., \( v \) is not redefined along the path).

*Example – Pattern Matching Program:*  
A method that searches for a pattern within a subject string can have multiple du-pairs:
- The variable `subject` is defined at the method entry and used to compute its length.
- The loop variable `iSub` is defined and updated, and its values are used in comparisons.
  
By enumerating all du-paths for each variable, we can define three main data flow coverage criteria:
- **All-Defs Coverage:** Each definition must reach at least one use.
- **All-Uses Coverage:** Each definition must reach all possible uses.
- **All DU-Paths Coverage:** Every possible def-clear path from each definition to each use must be executed.

### 4.3 Grouping DU-Paths

For clarity, du-paths can be grouped into:
- **Def-Path Set:** For a given definition at node \( n_i \), the set of all du-paths for variable \( v \) starting at \( n_i \).
- **Def-Pair Set:** For a given definition at \( n_i \) and a specific use at \( n_j \), the set of du-paths \( du(n_i, n_j, v) \).

These groupings help in systematically generating test cases that satisfy data flow criteria.



## 5. Generating Test Paths

### 5.1 From Structural Coverage

For node, edge, and edge-pair coverage, standard graph traversal algorithms (BFS/DFS) can generate test paths:
- **BFS** yields the shortest paths covering all nodes and edges.
- **Edge-Pair Coverage** can be generated by exploring all adjacent edge pairs in the CFG.

### 5.2 From Data Flow Coverage

Generating test paths for data flow criteria involves:
- Identifying all du-pairs for each variable.
- For **All-Defs Coverage,** ensuring each definition reaches at least one use.
- For **All-Uses Coverage,** ensuring each definition reaches every use.
- For **All DU-Paths Coverage,** generating test paths that cover every def-clear path from each definition to its uses.
- Often, side trips and detours are allowed (provided they remain def-clear) to cover as many du-paths as possible.

*Practical Example:*  
In the statistics program:
- A test case with a single-element array might yield a test path that covers one iteration of each loop, thereby covering du-paths for variables like `sum` and `mean`.
- A test case with a larger array (e.g., three elements) may produce longer test paths covering multiple iterations—thereby ensuring that definitions in the loops reach all possible uses.



## 6. Test Case Generation: Tools and Automation

While these analyses can be performed manually on small programs, automation is essential for real-world applications:
- **IDE Integration:** Many modern IDEs can generate CFGs automatically. Plugins are available for Eclipse, IntelliJ IDEA, and Visual Studio.
- **Graph Coverage Web Apps:** Tools such as the GMU Graph Coverage Webapp allow testers to input graphs and receive computed test requirements and test paths.
- **Symbolic Execution:** Tools like KLEE and Diffblue Cover can generate test cases that satisfy complex structural and data flow criteria by exploring paths symbolically.



## 7. Subsumption Relationships

Understanding how coverage criteria relate helps us choose an effective strategy:
- **Structural Coverage:**  
  - **Edge Coverage** subsumes **Node Coverage**.
  - **Edge-Pair Coverage** is stronger than Edge Coverage.
  - **Prime Path Coverage** subsumes both Node and Edge Coverage and often covers most du-paths.
  - **Round Trip Coverage** is a specialized form focused on cycles.
  
- **Data Flow Coverage:**  
  - **All Uses Coverage** subsumes **All-Defs Coverage**.
  - **All DU-Paths Coverage** is the most exhaustive, subsuming All Uses Coverage.
  - With certain assumptions (e.g., every use is preceded by a definition), achieving prime path coverage in a CFG often subsumes many data flow criteria as well.

These relationships guide us in designing test suites that are both comprehensive and efficient.



## 8. Challenges and Practical Considerations

Graph-based unit testing is powerful but comes with challenges:
- **Infeasible Paths:**  
  Complete path coverage is often infeasible, especially in loops. Testers must use heuristics or best-effort touring (allowing side trips/detours) to approximate full coverage.
- **Scalability:**  
  For large methods, the CFG can become very complex. Automated tools and symbolic execution are key to managing this complexity.
- **Optimal Test Suite Minimization:**  
  Finding the minimal set of test paths that cover all criteria is NP-complete. Heuristics and approximate methods are usually employed.



## 9. Case Studies and Practical Examples

### 9.1 Statistics Program

The statistics program computes the mean, median, variance, and standard deviation of an array:
- **CFG Derivation:**  
  The CFG is derived from initializations, two for loops, and print statements. Nodes represent basic blocks, and edges show loop iterations and branch exits.
- **Structural Coverage:**  
  Node and edge coverage can be achieved by a test path that covers one or more iterations of each loop.
- **Data Flow Coverage:**  
  DU-pairs for variables like `sum`, `mean`, and `i` are identified. Test cases with different array lengths (e.g., single-element vs. three-element arrays) expose different DU-paths.
- **Outcome:**  
  A fault (division by zero) is detected when an array of length 0 is used—a scenario uncovered by carefully analyzing DU-paths.

### 9.2 Pattern Matching Program

The pattern matching program finds the first occurrence of a pattern in a subject string:
- **CFG Derivation:**  
  The CFG captures initializations, a while loop with an inner for loop, and conditional statements.
- **Data Flow Augmentation:**  
  Definitions and uses are annotated for variables such as `subject`, `pattern`, `iSub`, and `rtnIndex`.
- **DU-Pair Analysis:**  
  Several DU-paths for each variable are identified and grouped into def-path and def-pair sets.
- **Outcome:**  
  The resulting analysis ensures that every definition of key variables reaches its use, helping to uncover subtle errors in pattern matching.



## 10. Conclusion and Reflection

In this chapter, we have delved deeply into unit testing based on graph models. We began by reviewing how code is abstracted into control flow and data flow graphs. Next, we explored a range of structural coverage criteria—from node and edge coverage to prime path and round trip coverage—and discussed how these criteria ensure that every critical part of the code’s execution is tested.

We then extended our analysis to data flow coverage, where the focus shifts to how variable values are defined and used. By generating du-paths and grouping them into def-path and def-pair sets, we can create rigorous test requirements that guarantee the proper propagation of data through the code.

Practical examples, such as the statistics and pattern matching programs, illustrate these concepts in a concrete manner. Along the way, we discussed the role of automation—from IDE-generated CFGs to symbolic execution tools—and acknowledged the inherent challenges in dealing with infeasible paths and optimal test suite minimization.

> *“Graph-based unit testing transforms the abstract world of code into a tangible map of execution paths. By illuminating every node, edge, and data dependency, we can systematically uncover hidden faults and build software that truly withstands the test of time.”*  


As you move forward, remember that these techniques are not just academic exercises; they are powerful tools in the ongoing pursuit of software quality. Whether you are manually analyzing a small method or using automated tools for large systems, the principles outlined in this chapter will guide you toward more effective and comprehensive testing.

