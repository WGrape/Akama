## Most Common Performance Issues in Parallel Programs

- Since the goal of Parallel Extensions is to simplify parallel programming, and the motivation behind parallel programming is performance, it is not surprising that many of the questions we receive about our CTP releases are performance-related.

- Developers ask why one program shows a parallel speedup but another one does not, or how to modify a program so that it scales better on multi-core machines.

- The answers tend to vary. In some cases, the problem observed is a clear issue in our code base that is relatively easy to fix. In other cases, the problem is that Parallel Extension does not adapt well to a particular workload. This may be harder to fix on our side, but understanding real-world workloads is a first step to do that, so please continue to send us your feedback. In yet another class of issues, the problem is in the way the program uses Parallel Extensions, and there is little we can do on our side to resolve the issue.

- Interestingly, it turns out that there are common patterns that underlie most performance issues, regardless of whether the source of the problem is in our code or the user code. In this blog posting, we will walk though the common performance issues to take into consideration while developing applications using Parallel Extensions.

**Amount of Parallelizable CPU-Bound Work**

- The number one requirement for parallelization is that the program must have enough work that can be performed in parallel. If only half of the work can be parallelized, Amdahl’s Law dictates that we are not going to be able to speed up the program by more than a factor of two.

- Also, additional CPUs thrown at a task will help the most if the CPU was the performance bottleneck. If the program spends 90% of its time waiting for a server to execute SQL queries, then parallelizing the program likely will not achieve significant benefits. (Well, you may still observe a speedup if there are multiple requests that we can send, say to multiple servers. In such cases, though, the asynchronous programming model would likely result in even better performance.)

- To benefit from parallelism, the total amount of processor-intensive work in a program must be large enough to dwarf the overheads of parallelization, and a large fraction of that work must be decomposable to run in parallel.

**Task Granularity**

- Even if a program does a lot of parallelizable work, we must be careful to ensure that we will split up the work into appropriately-sized chunks which will execute in parallel. If we create too many chunks, the overheads of managing and scheduling the chunks will be large. If we create too few chunks, some cores on the machine will have nothing to do.

- In some parts of the Parallel Extensions API, such as Parallel.For and PLINQ, the code in our library is responsible for deciding on the proper granularity of tasks. In other parts of the API, such as tasks and futures, it is the responsibility of the user code. Regardless of who is responsible for creating the tasks, appropriate task granularity is important in order to achieve good performance.

**Load Balancing**

- Even if there is enough parallelizable CPU work to make parallelism worthwhile, we need to ensure that the work will be evenly distributed among cores on the machine. This is complicated by the fact that different “chunks” of work may differ widely in the time required to execute them. Also, we often don’t know how much work each chunk will require until we execute it till completion.

- For example, if Parallel.For were to assume that all iterations of the loop take the same amount of time, we could simply divide the range into as many contiguous ranges as we have cores, and assign each range to one core. Unfortunately, since the work per iteration may vary, it is possible that one core will end up with many expensive iterations, while other cores will have less work to do. In an extreme situation, one core may end up with nearly all work, and we are back to the sequential case.

- Fortunately, our implementation of Parallel.For provides load balancing that should work well for most workloads. But, you are likely to encounter the load-balancing problem when writing your own concurrent code.

**Memory Allocations and Garbage Collection**

- Some programs spend a lot of time in memory allocations and garbage collections. For example, programs that manipulate strings tend to allocate a lot of memory, particularly if they are not designed carefully to prevent unnecessary allocations.

- Unfortunately, allocating memory is an operation that may require synchronization. After all, we need to ensure that memory regions allocated by different threads will not overlap.

- Perhaps even more seriously, allocating a lot of memory typically means that we will also need to do a lot of garbage collection work to reclaim memory that has been freed. If the garbage collection dominates the running time of your program, the program will only scale as well as the garbage collection algorithm.

- It is possible to mitigate this issue by turning on the server GC. See Chris Lvon’s blog post Server, Workstation and Concurrent GC for more information about how server GC works and how to enable it.

**False Cache-Line Sharing**

- In order to explain this particular performance problem of parallel programs, let’s quickly review a few details about how caches work on today’s mainstream computers. When a CPU reads a value from the main memory, it copies the value to cache, so that subsequent accesses to that value are much faster. In fact, rather than just bringing in that particular value into cache, the CPU will bring in also nearby memory locations. It turns out that if a program read a particular memory location, chances are that it is going to read nearby values too. So, values are moved between main memory and cache in chunks called cache lines, typically of size 64 or 128 bytes.

- One problem that arises on machines with multiple cores is that if one core invalidates a particular memory location, the version of that memory location cached by another core gets invalidated. Then, the core with an invalid cached copy must go all the way to the main memory on the next read of that memory location. So, if two cores keep writing and reading a particular memory location, they may end up continuously invalidating each other’s caches, sometimes dramatically reducing the performance of the program.

- The trickiest part of the issue is that the two cores don’t even have to be writing into the same memory location. The same problem occurs also if they are writing into two memory locations that are on the same cache line (hence the term “false cache-line sharing”).

- Strangely, this problem turns up in practice fairly regularly. For example, a parallel program that computes the sum of integers in an array will typically have a separate intermediate result for each thread. The intermediate results are likely to be either elements in an array or fields in a class. And, in both cases, they would likely end up close in memory.

- There are various techniques to prevent false sharing, or at to at least make it unlikely: padding data structures with garbage data, allocating them in an order that makes false sharing less likely, or allocating them by different threads.

**Locality Issues**

- Sometimes modifying a program to run in parallel negatively affects locality. For example, let’s say that we want to perform an operation on each element on an array. On a dual-core machine, we could create two tasks: one to perform the operation on the elements with even indices and one to handle odd indices.

- But, as a negative consequence, the locality of reference degrades for each thread. The elements that a thread needs to access are interleaved with elements it does not care about. Each cache line will contain only half as many elements as it did previously, which may mean that twice as many memory accesses will go all the way to the main memory.

- In this particular case, one solution is to split up the array into a left half and a right half, rather than odd and even elements. In more complicated cases, the problem and the solution may not be as obvious, so the effect of parallelization on locality of reference is one of the things to keep in mind when designing parallel algorithms.

**Summary**

- We discussed the most common reasons why a concurrent program may not achieve a parallel speedup that you would expect. If you keep these issues in mind, you should have an easier time designing and developing parallel programs.