---
layout: default
title: Async syntax
nav_order: 1
parent: Rust async model
---

# Async syntax

---

## Async functions

{: .warning }
> `async` function are `!Unpin` by default. See [Pinning]({% link docs/the-rust-async-model/pinning.md %}) for more details.


Asynchronous functions in Rust must be marked using the `async` keyword.


```rust
/// A simple Rust asynchronous function
async fn compute() {
    // ...
}
```

You can run `async` function using the `await` keyword, provided you are within an asynchronous context. This effectively means that to call an `async` function, all the parents of the current function must be `async`, up to a single function in which the asynchronous runtime is created. This means that `async` code [tends to propagate in Rust](#function-coloring).

{: .note }
> Be aware that calls to `await` will block code execution at that point in the function until the task being `await`ed completes.

```rust
// This uses tokio to generate an async runtime inside our main function.
#[tokio::main]
async fn main() {
    // do some stuff

    compute.await();

    // do more stuff
}
```

In the above example, calling `compute.await` pauses the execution of `main` until `compute` returns. This means calling `async` functions in a sequenece will effectively result in *synchronous* execution. For your code to be truelly executed in parallel, you will have to take adavantage of library functions such as `join!` in [Tokio]({% link docs/using-3rd-party-libraries/tokio.md %}).

---

## Async closures

Closures can also be defined as `async`. This is particularly helpful when dealing with one-shot asynchronous code such as when calling library function like `join!`.

```rust
async fn conpute() {
    let closure1 = async {
        // ...
    };
    let closure2 = async {
        // ...
    };
    // closure1 and closure2 will be executed in parallel.
    let (result1, result2) = tokio::join!(closure1, closure2).await;
}
```

Async closures might also need to *take ownership* of the data they use due to Rust ownership constraints. For that, we can use `async move`.

```rust
async fn compute() {
    let mut data = 1;
    // we can still access 'data' 
    // since the move hasn't occurred yet.
    println!(data);

    // data is 'captured' by this closure 
    // and can no longer be used outside of this scope.
    let closure = async move {
        data += 1;
        println!("{data}");
    };

    // it would be illegal to access 'data' at this point,
    // since it is has been moved into the above closure.
    // println!(data);

    closure.await;
}
```

{: .note }
> `async` closure cannot accept arguments: `async |...| {}` is not a valid syntax.

---

## Function coloring

Async syntax in Rust follows the principle of [function colorring](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/), that is to say: functions marked with `async` are fundamentally incompatible with non-`async` functions, and have to be called in special ways using `await`. This tends to lead to the uncontrolled porpagation of `async` functions, with synchronous functions becoming `async` in order to be able to call other `async` functions. This is different from Go for example, which allows functions to perform asynchronous computations without altering their signature.

### Async in Rust

```rust
// Note that if we were to change this function to be synchronous
// this would heavily change the syntax needed for calling this function.
async fn compute(url: &str) -> Result<String, _> {
    ...
}

// Async functions tend to propagate: we need to color 
// `main` as async in order to run other async functions
#[tokio::main]
async fn main() {
    let url = "https://example.com";
    match compute(url).await {
        Ok(content) => { ... },
        Err(e) => { ... },
    }
}
```

### Async in Go

```go
package main

// Go handles async operations through channels and goroutine.
// Channels can be passed to function as arguments. This does
// not introduce function coloring since the same function 
// decleration syntax is used as for non-async functions.
//
//      channels are messaging primitives responsible
//      for passing data and errors between threads
//                            \/
func compute(url string, ch chan<- string) {
    // ...
}

// Async code can be run from inside normal functions
// without having to change their signature.
func main() {
	url := "https://example.com"
	ch := make(chan string) // Create a channel for string type
	go fetchURL(url, ch) // Start the goroutine
	content := <-ch // Receive the result from channel
	println("Fetched content:", content)
}
```

{: .highlight }
> This raises an important concern as to wether or not simpler async models are viable for low-level systems programming.
