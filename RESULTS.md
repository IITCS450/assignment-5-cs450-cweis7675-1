# CS450 Assignment 5: User-Level Threads + Mutex
Name: Carson Weis

## Context-Switching Approach
My context-switching mechanism relies on cooperative scheduling using the "thread_yield()" function. When a thread 
decides to yield (or when it first starts), it calls into the scheduler. 

The actual context switch is handled by the assembly function in "uswtch.S". It works by:
1. Saving the current thread's callee-saved registers ("%ebp", "%ebx", "%esi", "%edi") onto its own stack.
2. Saving the current stack pointer ("%esp") into the old thread's context struct.
3. Loading the new thread's stack pointer from its context struct into "%esp".
4. Popping the previously saved registers off the new thread's stack.
5. Returning, which seamlessly resumes execution in the new thread's execution flow.

## Limitations of the Implementation
Because this is a simple, cooperative user-level threading library, there are a few inherent limitations:
* Maximum Threads: My thread table has a fixed size. The library supports a maximum of 64 concurrent threads (including the main thread). 
If "thread_create" is called when the table is full, it will fail.
* Stack Size: Each thread is allocated a fixed stack size of 4096 bytes (1 Page) using "malloc()"". Since there are no guard pages in this 
user-level implementation, a deep call stack or large local variables could cause a stack overflow and corrupt the heap.
* Cooperative Scheduling: There is no timer interrupt preemption. If a thread enters an infinite loop and forgets to call "thread_yield()", 
the entire process will hang, starving all other threads.