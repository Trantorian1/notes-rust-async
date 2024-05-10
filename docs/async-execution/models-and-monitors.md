---
layout: default
title: Models and Monitors
nav_order: 1
parent: Async Execution
---

# Models and Monitors

{: .def-title }
> thread monitor
>
> Code that is responsible for the handling, ordering and execution of tasks.

Let's explore some common models behind which executors are built.

---

## Queue-polling model

Consider the following: given *N* threads, each able to execute a single task at the same time, what happens if more than *N* tasks are scheduled? A solution to this problem is to store each `async` operation in a queue:

1. An unbounded queue is created for each available thread.
2. New `async` tasks are added to the least congested queue in FIFO (**F**irst **I**n **F**irst **O**ut) order.
3. When a task finishes executing, the next task in that queue is polled to be run, and so on.

{: .note }
> This has the issue that if a queue is empty, the remaining workload will not be balanced across threads.

---

## Work-stealing model

Adding new tasks to the least congested queue is not enough to balance the workload across multiple threads, since we do not know how long each task might last. This means that, while each thread queue might have the same number of tasks, one queue might complete (become empty) before the rest.

A solution to this problem is to allow empty queues to **steal** work from other threads by polling tasks from the most congested queue. In this way, work is balanced across threads even if a queue were to complete earlier.

{: .note }
>  We have so far assumed `async` tasks will complete in one go: this is not necessarily the case, as tasks might need to wait on external condition such as IO events before being able to make further progress. This can block the execution of new tasks.

---

## Yield compute model

In reality, asynchronous tasks might execute in steps as they wait for certain conditions to be met, such as IO availability or a network message. During that time, we are not actually performing any computation, and the current tasks is blocking it's running thread from making any progress. This is referred to as **starving**. How do we avoid this?

A solution to this problem is to have stalling tasks **yield** while they wait for certain conditions to be able to make progress in their computation. That is to say, a yielding task will be removed from the thread pool and won't perform any compute. When the required conditions are met, that task will be inserted back into the executor to be polled at a later time. This repeats until the task completes its computation.

{: .note }
> But how do we know which task to insert back into the executor? For that we need a way of identifying individual tasks.

---

## Waker model

Closely associated to the concept of yielding tasks is that of a **waker**.

{: .def-title }
> waker
>
> A structure which is part of an *asynchronous context*, and can be called to add a yielding task back into the executor to be polled again at a later time.

{: .def-title }
> asynchronous context
>
> A structure which contains identifying information about a specific `async` task. A unique *context* is passed to every `async` task that is run by the executor.

This way, once a task *yields* compute and the conditions required for it to resume are met, it can be identified and added back to the executor by calling it's associated **waker**, allowing it to make progress in its computation. This process repeats until the task finishes computing.

*But* **how does this work** 😵‍💫?  *If an `async` task* **yields**, *it is no longer executing code so as not to block other tasks. So what is performing the check to know when to* wake *that task? For that, we have to use* [Event callbacks]({% link docs/async-execution/event-callbacks.md %}).
