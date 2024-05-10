---
layout: default
title: Rust async model
nav_order: 3
has_children: true
permalink: /docs/rust-async-model
---

# The Rust async model

The Rust `async` model is designed with zero-cost abstractions in mind. This means that Rust can provide higher-level asynchronous programming concepts such as streams and ready-made executors with no added runtime cost as compared to coding these implementation by hand. In particular, these solutions are engineered in such a way as not to be slower than relying on their lower-level counterparts. This is done by trading greater compile-time complexity for better runtime performance.

In practice, this means that you do not have to handle low-level implementations of asynchronous logic and can instead focus on the algorithms themselves. This differs for example from C which relies much more on the developer implementing their own `async` execution logic.

{: .note }
> This however comes at the cost of obfuscation: while higher-level abstractions can lead to easier develoment, they also significantly raise the bar for entry into the `async` world.
