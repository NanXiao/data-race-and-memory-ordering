# Data race & memory ordering
The following definition is excerpted from [Is Parallel Programming Hard, And, If So, What Can You Do About It?](https://mirrors.edge.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.html):  

> Any situation in which one thread might be storing a value to a given variable while some other thread either loads from or stores to that same variable is termed a “data race”. 

Since compilers can do magic optimizations, CPUs can reorder instructions, among others, for "data race" context, i.e., except the shared variables are only loaded by multiple threads, synchronization must be used to access the shared variables (e.g., lock, atomic, etc.). Otherwise you may bump into a very weird bug one day, Boo!   

Why can't multiple-load of shared varible incur issue? The answer is from [Atomic weapons (part2)](https://www.youtube.com/watch?v=KeLBd2EJLOU):  

> The system must never invent a write to a variable that wouldn’t be written to in an SC execution.

Wait, what is `SC`? The answer is from [Atomic weapons (part1)](https://www.youtube.com/watch?v=A8eCGOqgvH4):  

> Sequential consistency: the result of any execution is the same as if the reads and writes occurred in some order, and the operations of each individual processor appear in this sequence in the order specified by its program.  

Another concept which is realted to `SC`:  

> Sequential consistency for data race free programs (SC-DRF, or DRF0): Appearing to execute the program you wrote, as long as you didn’t write a race condition.  

`SC-DRF` will define `memory model`:  
> You correctly synchronize your program (no race conditions), and the system provides the illusion of executing the program you wrote.  

`Memory model` leads to `memory ordering`. To fully understand `memory ordering`, `acquire` and `release` need to be defined first (From [Acquire and Release Semantics](https://preshing.com/20120913/acquire-and-release-semantics/)):  

> Acquire semantics is a property that can only apply to operations that **read** from shared memory, whether they are read-modify-write operations or plain loads. The operation is then considered a **read-acquire**. Acquire semantics prevent memory reordering of the read-acquire with any read or write operation that **follows** it in program order.  

> Release semantics is a property that can only apply to operations that **write** to shared memory, whether they are read-modify-write operations or plain stores. The operation is then considered a **write-release**. Release semantics prevent memory reordering of the write-release with any read or write operation that **precedes** it in program order.

Then it is easy to understand [std::memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order). `memory_order_seq_cst` is the most strongest one, which may introduce performance penalty. `memory_order_acquire` and `memory_order_release` can be used to synchronize among different threads: one thread calls `write-release`, the other thread calls `read-acquire` and sees the modification the first thread has done before `write-release`. `memory_order_acq_rel` is the combination of `memory_order_acquire` and `memory_order_release`. `memory_order_relaxed` is the weakest, and it is used for there is no synchronizaion need among threads (e.g., counter program).  

Another thing which always makes people confused is `volatile`. The following is excerpted from [Modern C](https://modernc.gforge.inria.fr/):  

> Takeaway 3.17.5.9 volatile objects are reloaded from memory each time they are accessed.  
> Takeaway 3.17.5.10 volatile objects are stored to memory each time they are modified.

The following is excerpted from [Is Parallel Programming Hard, And, If So, What Can You Do About It?](https://mirrors.edge.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.html):  
> the volatile keyword can prevent load tearing and store tearing in cases where the loads and stores are machine-sized and properly aligned. It can also prevent load fusing, store fusing, invented loads, and invented stores. However, although it does prevent the compiler from reordering volatile accesses with each other, it does nothing to prevent the CPU from reordering these accesses. Furthermore, it does nothing to prevent either compiler or CPU from reordering non-volatile accesses with each other or with volatile accesses.

To summarize, `volatile` prevents compiler to do optimization, but it can't prevent CPU to reorder instruction, nor guarantee atomic access. However, `volatile` can literally be used in some scenarios to avoid using atomic access: e.g., one thread stores machine-sized, properly aligned data while another thread loads.


## References:  
[Acquire and Release Semantics](https://preshing.com/20120913/acquire-and-release-semantics/)  
[Atomic types should be used instead of "volatile" types](https://rules.sonarsource.com/c/tag/c11/RSPEC-3687)  
Atomic weapons: [part 1](https://www.youtube.com/watch?v=A8eCGOqgvH4) and [part 2](https://www.youtube.com/watch?v=KeLBd2EJLOU)  
[Benign Data Races: What Could Possibly Go Wrong?](https://software.intel.com/en-us/blogs/2013/01/06/benign-data-races-what-could-possibly-go-wrong)  
[C11/C++11 Memory model. What is it, and why?](https://www.slideshare.net/MikaelRosbacke/c11c11-memory-model-whart-is-it-and-why)    
[C++ atomics, from basic to advanced. What do they really do?](https://www.youtube.com/watch?v=ZQFzMfHIxng)  
[C++11 introduced a standardized memory model. What does it mean? And how is it going to affect C++ programming?](https://stackoverflow.com/questions/6319146/c11-introduced-a-standardized-memory-model-what-does-it-mean-and-how-is-it-g)  
[C++11/14/17 atomics and memory model: Before the story consumes you](https://www.youtube.com/watch?v=DS2m7T6NKZQ)  
[Hacker News](https://news.ycombinator.com/item?id=11908098)   
[Is Parallel Programming Hard, And, If So, What Can You Do About It?](https://mirrors.edge.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.html)   
[Modern C](https://modernc.gforge.inria.fr/)  
[What do each memory_order mean?](https://stackoverflow.com/questions/12346487/what-do-each-memory-order-mean)  
[Who's afraid of a big bad optimizing compiler?](https://lwn.net/Articles/793253/)     
[You Don’t Know Jack about Shared Variables or Memory Models](https://queue.acm.org/detail.cfm?id=2088916)  
[簡介 C++11 atomic 和 memory order](https://medium.com/fcamels-notes/%E7%B0%A1%E4%BB%8B-c-11-memory-model-b3f4ed81fea6)  
