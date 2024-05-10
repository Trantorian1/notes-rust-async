---
layout: default
title: Models and Monitors
nav_order: 1
parent: Async Execution
---

# Models and Monitors

> 📚 A **thread monitor** is code that is responsible for the handling, ordering and execution of asynchronous code.

Let's explore some common models behind which executors are built.

---

## Queue-polling model

Consider the following: given *N* threads, each able to execute a single task at the same time, what happens if more than *N* tasks are scheduled? A solution to this problem is to store each `async` operation in a queue:

1. An unbounded queue is created for each available thread.
2. New `async` functions are added to the least congested queue in FIFO (**F**irst **I**n **F**irst **O**ut) order.
3. When a function finishes executing, the next function in that queue is polled to be run, and so on.

>  ⚠️ This has the issue that if a queue is empty, the remaining workload will not be balanced across threads.

## Work-stealing model

Adding new tasks to the least congested queue is not enough to balance the workload across multiple threads, since we do not know how long each task might last. This means that, while each thread queue might have the same number of tasks, one queue might complete (become empty) before the rest.

A solution to this problem is to allow empty queues to **steal** work from other threads by polling tasks from the most congested queue. In this way, work is balanced across threads even if a queue were to complete earlier.

>  ⚠️ We have so far assumed `async` tasks will complete in one go: this is not necessarily the case, as some tasks might wait for special events such as IO availability. This can block the execution of new tasks without doing any compute.

## Yield compute model

In reality, asynchronous tasks might execute in steps as they wait for certain dependencies to be met, such as IO availability or a network message. During that time, we are not actually performing any computation, and the current tasks is blocking it's running thread from making any progress. This is referred to as **starving**. How do we avoid this?

A solution to this problem is to have stalling tasks **yield** while they wait for certain conditions to be able to make progress in their computation. When these conditions are met, they are inserted back into the executor to be polled at a later time. This repeats until they complete their computation.

> ⚠️ But how do we know which task to insert back into the executor? For that we need a way of identifying individual tasks.

## Waker model

Closely associated to the concept of yielding tasks is that of a **waker**.

📚 a **waker** is a structure which is part of an *asynchronous context*, and can be called to add back a specific task back to the top of the queue, effectively force-polling it *.*

📚 an **asynchronous** **context** is a structure which contains identifying information about a specific `async` task. A unique *context* is passed to every `async` task that is run by the executor.

This way, once a task *yields* compute and the conditions required for it to resume are met, it can be identified and added back to the top of the queue by calling it's associated **waker**. This allows it to be polled again at completion once the current task finishes. This process can then repeat as further requirements for computation make themselves known.

> But **HOW DOES THIS WORK 😵‍💫**  If an `async` task *yields*, it is no longer executing code so as not to block other tasks. So what is performing the check to know when to *wake* that task? For that, we have to use [Event callbacks]({% link docs/async-execution/event-callbacks.md %}).
