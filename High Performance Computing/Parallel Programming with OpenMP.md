All constructs are used in C/C++ via the basic syntax: `#pragma omp <construct_name> [clauses]`

>[!Note]
 NOTE: OpenMP allows and encourages combining various pragmas to avoid manually blocking loops or defining complex boilerplate, improving performance and readability. Common examples include:
>* `parallel for` 
>* `target teams`
>* `distribute simd`
>* `target teams distribute parallel for`

| Category                      | Pragma           | Brief Description                                                                                                                                                                                               | Example                                                                |
| ----------------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Parallelism Management**    | `parallel`       | Defines a parallel region (fork/join model). The master thread generates a team of threads, and the code within the block is replicated and executed by all threads.                                            | Pi?                                                                    |
| **Worksharing**               | `for`            | Distributes the iterations of a `for` loop (data parallelism) among the active threads in the team. Includes an implicit synchronization barrier at the end, unless the `nowait` clause is used.                | [[2. openMP - Advanced#Exercise 1 - Pi\|Pi]]                           |
|                               | `simd`           | (NOT PRESENTED IN THEORY SLIDES). Used on `for` loops, tells the compiler to transform that loop in a SIMD loop. This exploits the processor's vectorial operations to execute multiple iterations at once.     | [[2. openMP - Advanced#Exercise 3 - Jacobi#Slide solution(s)\|Jacobi]] |
|                               | `sections`       | Outlines a block of code containing independent tasks (static Task Parallelism / MPMD) to be distributed among threads.                                                                                         |                                                                        |
|                               | `section`        | Used within the `sections` construct, it encloses a single independent block of code that will be executed in parallel with other `section` blocks.                                                             |                                                                        |
|                               | `single`         | Defines a section of code that must be executed by *only one thread* within the team, not necessarily the master. Includes an implicit barrier at the end.                                                      | Fib?                                                                   |
|                               | `master`         | Defines a section of code that must be executed *exclusively by the master thread*. Unlike `single`, it does not impose any implicit synchronization/barrier at the end.                                        |                                                                        |
| **Synchronization**           | `barrier`        | Explicit barrier: all threads of the executing team suspend their activities at this point until the entire team has reached the barrier.                                                                       | Jacobi?                                                                |
|                               | `critical`       | Outlines a critical section. Guarantees mutual exclusion: only one thread at a time can execute the contained code block, avoiding *race conditions*.                                                           | [[2. openMP - Advanced#Exercise 1 - Pi\|Pi]]                           |
|                               | `atomic`         | Ensures that the update (write/read) of a specific shared variable occurs atomically. Often has lower overhead compared to `critical`.                                                                          | [[2. openMP - Advanced#Exercise 1 - Pi\|Pi]]                           |
| **Tasking Model**             | `task`           | Packages a unit of work (code + data environment) whose execution can be deferred or executed immediately by a thread in the team. Perfect for unbounded loops (while), linked lists, and recursive algorithms. | [[2. openMP - Advanced#Exercise 2 - Fibonacci\|Fibonacci]]             |
|                               | `taskwait`       | Barrier for tasks: suspends the execution of the current task until all of its direct child tasks have completed.                                                                                               | [[2. openMP - Advanced#Exercise 2 - Fibonacci\|Fibonacci]]             |
| **Accelerators / Offloading** | `target`         | Transfers execution control from the host to the target device (e.g., GPU). By default, it also handles the mapping of required data (to and from the device).                                                  |                                                                        |
|                               | `target data`    | Maps variables to the target device's data environment without necessarily executing code. Variants include: `target enter data`, `target exit data`, and `target update`.                                      |                                                                        |
|                               | `declare target` | Instructs the compiler that the specified functions or variables must be compiled for the target device (so they can be invoked from within `target` regions).                                                  |                                                                        |
|                               | `teams`          | Creates a league of thread teams on the target device. Teams **cannot** synchronize with each other.                                                                                                            |                                                                        |
|                               | `distribute`     | Distributes the iterations of the associated loops in chunks across the *master threads* of each team (often used in conjunction with `teams`).                                                                 |                                                                        |
## Important Clauses
Clauses modify the behavior of the directives. Here are the main ones categorized by their purpose:
### Data Scoping Clauses
These clauses dictate how variables are shared or privatized among threads.
- **`shared(list)`**: All threads in the team share a single copy of the variables in the list.
- **`private(list)`**: Each thread gets its own uninitialized, private copy of the variables.
- **`firstprivate(list)`**: Similar to `private`, but the private copies are initialized with the original value the variable had before entering the parallel region.
- **`lastprivate(list)`**: Similar to `private`, but the value from the logically last iteration (or last section) is copied back to the original variable after the parallel region.
- **`default(shared | none)`**: Sets the default data-sharing attribute. Using `default(none)` forces the programmer to explicitly define the scope for every variable, which is highly recommended for safety.
- **`reduction(op : list)`**(As used in [[2. openMP - Advanced#Exercise 1 - Pi|Pi]]): Instructs the compiler to create private copies of the variables for every thread. At the end of the region, the partial results are safely combined into the original shared variable using the specified operator (e.g., `+`, `*`).
### Loop Scheduling Clauses (used with `for`)
These clauses determine how iterations are partitioned among threads.
- **`schedule(static[, chunk])`**: Iterations are divided into blocks of size `chunk` and assigned to threads in a round-robin fashion. Best for simple, regular loops with balanced workloads.
- **`schedule(dynamic[, chunk])`**: Threads dynamically grab blocks of iterations of size `chunk` from a work queue. Excellent for unbalanced workloads where iterations take varying amounts of time.
- **`schedule(guided[, chunk])`**: Similar to dynamic, but the block size starts large and exponentially shrinks down to `chunk` as the calculation proceeds.
- **`schedule(runtime)`**: The scheduling type and chunk size are deferred until runtime and taken from the `OMP_SCHEDULE` environment variable.
- `collapse(n)` (As used in [[2. openMP - Advanced#Exercise 3 - Jacobi#Lecture solution|Jacobi]]): where $n$ is the number of nested `for` loops. The collapse clause fuses the nested loops into a single much longer loop. This makes for a much better workload balancing between worker threads. One flaw is that `collapse()` makes it nearly impossible to know what is going on at any given iteration.
### Synchronization & Execution Clauses
- **`nowait`**: Removes the implicit barrier at the end of worksharing constructs (like `for`, `sections`, or `single`), allowing threads to proceed to the next block without waiting for others.
### Tasking Clauses (used with `task`)
- **`depend(type: list)`**: Defines a task data-flow execution model. It forces constraints based on data dependencies.
    - `in`: The task will depend on previously generated tasks that reference at least one list item in an `out` or `inout` clause.
    - `out` / `inout`: The task must complete before subsequent tasks that reference the item can start.
### Device and Mapping Clauses (used with `target` and `teams`)
- **`map([map-type:] list)`**: Defines how data is moved between the host and the target device. Types include:
    - `to`: Copy from host to device.
    - `from`: Copy from device to host at the end.
    - `tofrom`: Copy both ways (default behavior in many cases).
    - `alloc`: Allocate uninitialized memory on the device.
    - `release` / `delete`: Manage memory deallocation.
- **`num_teams(expr)`**: Specifies the number of teams to generate in a league.
- **`thread_limit(expr)`**: Specifies the maximum number of threads allowed per team.