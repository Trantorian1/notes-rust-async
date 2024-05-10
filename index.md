---
title: Home
layout: home
nav_order: 1
permalink: /
---

# Welcome to Async Rust!

{: .warning }
> This is a personal note guide designed for the study of asynchronous Rust, and as such it may contain some invonluntary errors. Use at you own risk!

The combination of Rust's low-level architecture as well as memory safety and ownership model make it an excellent candidate for *easily* writing *safe(r)* multihreaded code:

* Rust being low-level allows for excellent runtime multithreading (no garbage collector).
* Rust's ownership model makes it very difficult to introduce data races into a multithreaded environment.
* Rust's [package ecosystem](https://crates.io/categories/asynchronous) has excellent support for asynchronous paradigms.

---

## General defenitions

{: .def-title }
> asynchronous code
>
> Describes code that can execute in a *non-blocking* way and *out-of-order*. This is especially important to keep in mind in the context of Rust ownership since `async` functions can theoretically live for as long as the program itself, so any data used must be *owned* by that function.

{: .def-title }
> lock
>
> A low-level asynchronous primitive used to synchronize data across threads. There are several types of locks in Rust, such a `Mutex` and `RwLock`.

{: .def-title }
> executor
>
> An environment used to run `async` code. Simply put it is a set of code, often library-generated, which is responsible for running your `async` functions and handling their distribution across multiple hardware threads.

{: .def-title }
> hardware threads
>
> An os-level primitive for handling concurrency. When you create a new hardware thread, you are creating a 1:1 copy of the stack which then lives independently and simultaneously to the rest of your program. *Hardware threads are computationally expensive*, which is why we generally refer to techniques such as [thread monitors](https://en.wikipedia.org/wiki/Monitor_(synchronization)) to distribute *many* `async` operations across *few* hardware threads.

## Rust-specific definitions

{: .def-title}
> future
>
> An aynchronous computation to be completed at a future time.

{: .def-title }
> task
>
> A unit of compute in an **executor**, responsible for running a **future** and managing it's state.

---

## Some things to keep in mind

Rust is *not* magic. 
What I mean by that is that -like any other programming language- Rust will not save you from yourself. It is *still* possible to introduce data races and it *still* possible to engineer new and creative asynchronous **runtime** exceptions. Where Rust's truly shines is in it's ability to let let you focus on algorithms instead of the implementation of low-level primitives: existing abstractions allow you to focus on the problem at hand, instead of having to deal with already-solved problems. Think of Rust in this context as a mix between high-level abstractions while maintaining low-level performance.

{: .note }
> This still comes at the cost of a higher learning curve, especially when considering lifetimes (which can be quite a hindrance in an multithreaded environment).

---

## Sources

This repo is largely based on the [async rust book](https://rust-lang.github.io/async-book/), as well as some personal research.
