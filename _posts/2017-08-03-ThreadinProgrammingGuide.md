---
title: Threading Programming Guide
description: Threading Programming Guide
header: Threading Programming Guide
---

Nature, time, and patience are the three greatest physicians.

##Threading Terminolgy
The term *thread* is used to refer to a separate path of execution for code.
The term *process* is used to refer to a running executable, which can encompass multiple threads.
The term *task* is used to refer to the abstract concept of work that needs to be performed.

At the application level, all threads behave in essentially the same way as on other platforms.After starting a thread, the thread runs in one of three main states: running, ready, or blocked.if a thread is not currently running, it is either blocked and waitting for input or it is ready to run but not scheduled to do so yet.The thread continues moving back and forth among these states until it finally exits and moves to the terminated state.

##Run Loops
A run loop is a piece of infrastructure used to manage events arriving asynchronously on a thread.A run loop works by monitoring one or more event source for the thread. As events arrive, the system wake up the thread and dispatch the events to the run loop, which then dispatches them to the handlers you specify. if no events are present and ready to be handled, the run loop put thread to sleep.

##Synchronization Tools
One of the hazards of the threaded programming is resource contention among multiple threads.If multiple threads try to use or modify the same resource at the same time, problems can occour. One way to alleviate the problem is to eliminate the shared resource altogether and make sure each thread has its own didtinct set of resource on which to operate. When maintaining completely separate resources is not an option though, you may have to synchronize access to the resouce using blocks, conditions, atomic operations, and other techniques.

The most common type of lock is mutual exclusion lock, also as known as a *mutex*.

Atomic operations offer a lightweight alternative to locks in situations where you can perform mathematical or logical opertaions on scalar date type.Atomic operations use special hardware instructions to ensure that modifications to a variable are completed before other threads have a chance to access it.

##Inter-thread Communication
Direct messaging, global variables, shared memory, and objects, conditions, run loops sources, ports and sockets, message queues,cocoa distributed objects.