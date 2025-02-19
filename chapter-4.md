# Chapter 4: Software Design and Integration Testing

*“In software engineering, integration testing is not merely the assembly of pre‐tested modules. It is the rigorous examination of the interfaces, data exchanges, and control flows that bind the modules together. Graph‐based models provide us with a powerful abstraction to reason about and verify these interconnections.”*  
— Professor Meenakshi D’Souza



## 1. Introduction

Integration testing marks the phase when individual, unit‐tested modules are combined to form a larger, cohesive system. Unlike unit testing—which focuses on internal behavior of a single module—integration testing concentrates on the interactions among modules, emphasizing the correctness of interfaces, data exchanges, and sequencing of calls.

In this chapter, we explore how graph‐based testing techniques, which we developed for unit testing, extend naturally to integration testing. We will discuss:

- How software design partitions a system into modules with well‐defined interfaces.
- The role of integration testing in verifying module interactions.
- Graph models (call graphs, FSMs, and sequencing graphs) used to represent integration aspects.
- Structural and data flow coverage criteria applied at the integration level.
- Integration testing techniques (incremental, top‐down, bottom‐up, sandwich, and big bang).
- The use of test stubs and test drivers.
- Examples illustrating these concepts.



## 2. Software Design, Modules, and Interfaces

### 2.1 Modules and Design

Software design organizes a system into self‐contained units or modules. A module can be a single function, a method, a class, or even a subsystem. The design document (or module hierarchy) specifies how these modules interact.

### 2.2 Interfaces

Modules communicate through interfaces that define the mechanism for passing control and data. Common interface types include:

- **Procedure Call Interface:** One module calls another (e.g., a method call). Parameters (actual and formal) are exchanged, and control transfers back upon completion.
- **Shared Memory Interface:** Multiple modules read from and write to a common memory block or global variable.
- **Message-Passing Interface:** Modules exchange information through messages over queues or channels (typical in client-server systems).

Interface errors are significant—empirical studies estimate that up to 25% of software faults are due to interface mistakes such as:
- Incorrect or misinterpreted parameter passing.
- Inadequate error handling.
- Initialization and synchronization issues.



## 3. Integration Testing Fundamentals

Integration testing is performed after unit testing, as modules (or components) that have been verified individually are combined incrementally. Its primary focus is on:

- **Testing Interfaces:** Ensuring that the data and control flow across module boundaries is correct.
- **Validating Interactions:** Checking that modules, when integrated, behave as specified.
- **Handling Incomplete Systems:** Using test stubs (for missing callees) and test drivers (for missing callers) to simulate parts of the system.

### 3.1 Integration Testing Techniques

There are several strategies for integration testing:

- **Incremental Integration Testing:** The system is built and tested module by module.  
  - *Top-Down:* Begin with the top-level module and gradually integrate lower-level modules (using stubs for missing ones).  
  - *Bottom-Up:* Start with the lowest-level modules and work upward (using drivers to simulate higher-level calls).
- **Sandwich Testing:** A hybrid approach that combines top-down and bottom-up integration.
- **Big Bang Testing:** All modules are integrated at once and tested as a whole (riskier and harder to diagnose).

Test stubs and drivers are essential scaffolding components that allow testing even when some modules are incomplete.



## 4. Graph Models for Integration Testing

Graph models provide a visual and analytical framework for integration testing. At the integration level, the graph models differ from unit-level CFGs in that:

- **Nodes** represent entire modules, methods, or components (including stubs and drivers).
- **Edges** represent call interfaces—i.e., the act of one module invoking another.

These graphs are often referred to as **call graphs**. In addition, for design-level testing, we sometimes use graphs that capture sequencing constraints and even finite state machine (FSM) models for behavioral specifications.

### 4.1 Structural Graph Coverage Criteria

For call graphs, the structural coverage criteria focus on:
- **Node Coverage:** Every module (node) is invoked at least once.
- **Edge Coverage:** Every call interface (edge) is exercised at least once.
- **(Specified) Path Coverage:** Particular sequences of module calls are tested, ensuring that the order of interactions (the “flow” among modules) satisfies design specifications.

Unlike unit testing, prime path coverage is less applicable here because the emphasis is on the correctness of interfaces rather than the internal looping structure of a module.

### 4.2 Data Flow Graph Coverage Criteria at the Integration Level

When modules interact, data is passed between them. Testing these interactions requires us to extend data flow concepts:
- **Coupling Variables:** Variables passed from one module (caller) to another (callee). In parameter coupling, an actual parameter (e.g., variable *x* in module A) is associated with a formal parameter (e.g., variable *y* in module B). In shared memory interfaces, the same variable is accessed by different modules.
- **Last-Definition and First-Use:** At the call interface, the *last definition* of a variable in the caller (or callee before return) and the *first use* in the callee (or caller after return) are of critical importance.
- **Coupling DU-Pairs:** A du-pair across modules (coupling du-pair) is defined from the last definition in the caller to the first use in the callee. Data flow coverage criteria can be applied to these du-pairs:
  - **All-Coupling-Defs Coverage:** Each last definition must reach at least one first use.
  - **All-Coupling-Uses Coverage:** Each last definition must reach every possible first use.
  - **All-Coupling-DU-Paths Coverage:** Every simple, def-clear path from the last definition to every first use must be executed.



## 5. Applying Graph Coverage Criteria to Integration Testing

### 5.1 Structural Testing of Integration

In integration testing using call graphs:
- **Step 1:** Identify the module hierarchy (or call graph).  
  For example, a module hierarchy document might show modules A, B, C, and D where:
  - Module A calls B, C, and D.
  - Module C further calls modules E, F, and G.
- **Step 2:** Generate test requirements:
  - *Node Coverage:* Ensure every module (or its test stub/driver) is called.
  - *Edge Coverage:* Ensure every interface (call edge) between modules is traversed.
  - *Path Coverage (if applicable):* Test specific sequences of calls to validate interaction order.
- **Step 3:** Create test cases (using stubs or drivers as necessary) that exercise these requirements.

### 5.2 Data Flow Testing of Integration

For data flow testing at integration:
- **Step 1:** Identify coupling variables at call sites.  
  For instance, if module A calls module B passing variable *x* (actual parameter) which becomes *y* (formal parameter), then treat (*x, y*) as a coupling pair.
- **Step 2:** Determine the *last definition* in the caller before the call and the *first use* in the callee after the call.
- **Step 3:** Generate coupling DU-pairs and then du-paths.  
  Create test cases that exercise each coupling du-path to ensure that the expected data value flows correctly across the interface.
- **Step 4:** If necessary, allow side trips (provided they are def-clear) to cover complex interactions.

*Example – Quadratic Root Calculation:*  
Consider a system where the main module calls a quadratic root calculator:
- **Caller (Main):** Reads values X, Y, Z and passes them as parameters.
- **Callee (Root):** Accepts parameters A, B, C and computes roots, returning a Boolean flag.  
  Coupling occurs in two ways:
  - **Parameter Coupling:** X is coupled with A, Y with B, and Z with C.
  - **Return Value Coupling:** The callee’s computed result (and any global shared variables such as Root1 and Root2) are passed back.
  
Test cases must be devised so that the last definitions of X, Y, and Z (just before the call) and the first uses in Root are correctly exercised. Similarly, the return values must be verified at the call site.

### 5.3 Sequencing Constraints and State Behavior

Integration testing also requires verifying sequencing constraints:
- **Sequencing Constraints:** Rules that impose the order in which methods must be called.  
  For example, a File ADT may have the following constraints:
  - *open(f)* must be called before any *write(t)*.
  - *open(f)* must be called before *close()*.
  - *write(t)* must not occur after *close()* without an intervening *open(f)*.
- **Testing Sequencing Constraints:**  
  - **Static Analysis:** Derive the control flow graph (CFG) of the application that uses the File ADT and verify that paths exist (or do not exist) which violate the sequencing rules.
  - **Dynamic Testing:** Execute test cases that intentionally attempt to violate the constraints to see if proper error handling occurs.

For sequencing constraints that involve counting (e.g., ensuring at least as many *enQueue()* calls as *deQueue()* calls in a Queue), a finite state machine (FSM) model can be more appropriate. In FSM models, states represent the number of elements in the queue, and transitions correspond to calls to *enQueue()* and *deQueue()*.



## 6. Finite State Machines (FSMs) for Design Specifications

FSMs provide a framework for modeling the behavior of systems at the design specification level. They are particularly useful for capturing:
- **State Behavior:** The various states a system can be in (e.g., open/closed door for an elevator, empty/partial/full queue).
- **Transitions:** How the system moves from one state to another based on events, conditions, or method calls.
- **Guards and Actions:** Each transition may have a guard (a condition that must be true) and an action (an operation that is performed).

### 6.1 FSM Coverage Criteria

When using FSMs for integration testing:
- **State (Node) Coverage:** Every state must be visited at least once.
- **Transition (Edge) Coverage:** Every transition must be executed.
- **Transition-Pair Coverage:** Every pair of consecutive transitions is executed, which may reveal sequencing issues.
- **Data Flow in FSMs:** Although FSMs typically do not include detailed def-use information, guards and actions can be analyzed to ensure that the correct data values trigger state transitions.

*Example – Elevator Door FSM:*  
A simple FSM may model an elevator door with states such as *Closed* and *Open*. Transitions are triggered by events (e.g., pressing the open button) provided that conditions (e.g., elevator speed is 0) are met. Testing the FSM involves designing sequences of events that exercise every state and transition.



## 7. Integration Testing Process and Practical Considerations

### 7.1 Process Overview

1. **Model Construction:**  
   – Derive graph models (call graphs, FSMs, sequencing graphs) from design documents, source code, or specifications.

2. **Test Requirement Generation:**  
   – Apply structural and data flow coverage criteria to the graph models.  
   – Identify nodes (modules), edges (interfaces), and coupling du-pairs (for data flow).

3. **Test Case Generation:**  
   – Use manual analysis and automated tools (IDEs, symbolic execution, graph coverage web apps) to generate test cases that satisfy the coverage criteria.  
   – Develop test stubs or drivers when modules are incomplete.

4. **Execution and Evaluation:**  
   – Execute test cases, monitoring for violations of sequencing constraints, interface errors, and state transitions.  
   – Use both static analysis (graph-based analysis) and dynamic testing to evaluate compliance.

### 7.2 Practical Considerations

- **Handling Infeasible Paths:**  
  When paths (or du-paths) are infeasible due to design constraints, best-effort touring (allowing side trips that remain def-clear) is employed.

- **Tool Support:**  
  Integration testing benefits from automated graph generation tools, test data generation using symbolic execution, and coverage analysis tools that can measure both structural and data flow coverage.

- **Certification Standards:**  
  In critical domains (e.g., aerospace, as mandated by DO-178C), integration testing must rigorously verify data coupling and control coupling. Graph-based techniques provide a formal basis for such certification.



## 8. Classical Coverage Criteria Revisited

While our discussion has focused on graph-based integration testing, it is instructive to relate these modern approaches to traditional coverage criteria:
- **Statement Coverage ≡ Node Coverage:**  
  Executing every module (or basic block) is analogous to statement coverage.
- **Branch Coverage ≡ Edge Coverage:**  
  Testing every call interface (or branch in a CFG) reflects branch coverage.
- **Basis Path Testing:**  
  Cyclomatic complexity, defined as \( M = E - N + 2P \), gives the number of linearly independent paths. Basis path testing ensures that each independent path is exercised.
- **Decision-to-Decision (DD) Paths:**  
  DD-paths are paths between decision points in the CFG. Although less common in integration testing, they provide insight into the sequencing of decisions.

Understanding these relationships reinforces that integration testing is an extension of unit testing—expanding the scope from isolated code to the interactions between modules.



## 9. Case Studies

### 9.1 Quadratic Root Calculator

- **Design:**  
  A main module calls a function to compute quadratic roots, passing parameters (X, Y, Z) and receiving a Boolean result along with global variables for roots.
- **Call Graph Analysis:**  
  Nodes represent the main method and the quadratic function. Edges represent the call interface.  
- **Data Flow:**  
  Coupling occurs through parameter passing (X→A, Y→B, Z→C) and return value coupling (result, Root1, Root2).
- **Testing:**  
  Test cases must verify that the last definitions in the caller are correctly transmitted to the callee and that the first uses in the callee yield the correct output.

### 9.2 File Abstract Data Type (File ADT)

- **Design:**  
  The File ADT provides methods such as *open(f)*, *write(t)*, and *close()*.
- **Sequencing Constraints:**  
  - *open(f)* must precede *write(t)* and *close()*.  
  - A *write(t)* cannot occur after *close()* unless a new *open(f)* occurs.
- **Graph-Based Testing:**  
  Control flow graphs are derived from sample code that uses File ADT. Test requirements are generated to check that every path from an *open()* to a *close()* includes at least one *write()*, and that no *write()* occurs after a *close()* without a new *open()*.
- **Result:**  
  By examining the CFG, paths that violate these sequencing constraints are identified, enabling the detection of interface errors.

### 9.3 Queue Integration

- **Design:**  
  A queue module offers methods *enQueue()* and *deQueue()*.  
- **FSM Modeling:**  
  With a fixed capacity (e.g., two elements), the queue’s states (empty, one element, full) can be modeled as an FSM.  
- **Sequencing Constraint:**  
  The number of *enQueue()* calls must be at least equal to the number of *deQueue()* calls.
- **Testing:**  
  The FSM is used to generate test cases that ensure the queue never transitions into an invalid state (e.g., de-queuing from an empty queue).



## 10. Conclusion and Reflections

Integration testing is the crucible where individually correct modules are united to form a working system. Graph-based testing techniques—whether applied to call graphs, sequencing constraints, or finite state machines—offer a structured, formal approach to verifying that module interactions are correct.

In this chapter, we have:

- Explained the role of modules, interfaces, and the overall design in software architecture.
- Introduced graph models tailored for integration testing, including call graphs and FSMs.
- Defined both structural and data flow coverage criteria in the context of integration testing.
- Outlined practical techniques (incremental, top‐down, bottom‐up, sandwich, and big bang) for integration testing.
- Discussed the importance of test stubs, test drivers, and coupling variables.
- Revisited classical coverage criteria and related them to modern graph-based approaches.
- Presented case studies (Quadratic Root Calculator, File ADT, Queue) that illustrate these concepts in a real-world context.

By integrating these graph-based techniques into the integration testing process, engineers can systematically verify that all module interfaces behave correctly and that the data flowing across these interfaces meets the design specifications. As you advance in your testing career, remember that a thorough understanding of these models not only aids in detecting errors but also contributes significantly to building robust, reliable software systems.

*“Integration testing is where the promise of modular design is fulfilled—or shattered. Through the lens of graph theory, we can illuminate the hidden dependencies and interactions that determine the overall system’s quality.”*  
— Professor Meenakshi D’Souza

