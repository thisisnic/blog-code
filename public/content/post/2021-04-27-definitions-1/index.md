---
title: 'Definitions #1'
author: ''
date: '2021-04-27'
slug: definitions-1
categories: []
tags:
  - arrow
---

This post contains answers to a number of questions I had when reading through the Apache Arrow dev mailing list.  In future posts I'll look up how to properly use footnotes in markdown!

## What is DataFusion?

> DataFusion is an attempt at building a modern distributed compute platform in Rust, leveraging Apache Arrow as the memory model.

## What is Ballista?

> Ballista is a distributed compute platform primarily implemented in Rust, powered by Apache Arrow.  

It has now been donated to the Arrow project and so development is happening there.

It is inspired by Apache Spark, but used Rust as the main execution language, is column-based (rather than row-based), and allows other advantages of Arrow to be exploited.

## What is ORC?

> Apache ORC (Optimized Row Columnar) is a free and open-source column-oriented data storage format of the Apache Hadoop ecosystem. It is similar to the other columnar-storage file formats available in the Hadoop ecosystem such as RCFile and Parquet. It is compatible with most of the data processing frameworks in the Hadoop environment. 

> ORC is a self-describing type-aware columnar file format designed for Hadoop workloads. It is optimized for large streaming reads, but with integrated support for finding required rows quickly. Storing data in a columnar format lets the reader read, decompress, and process only the values that are required for the current query. Because ORC files are type-aware, the writer chooses the most appropriate encoding for the type and builds an internal index as the file is written.

## What is Gandiva?

> By combining LLVM with Apache Arrow libraries, Gandiva can perform low-level operations on Arrow in-memory buffers such as sorts, filters, and projections that are highly optimized for specific runtime environments, improving resource utilization and providing faster, lower-cost operations of analytical workloads.

> Gandiva is a new open source project licensed under the Apache license and developed in the open on GitHub. It is provided as a standalone C++ library for efficient evaluation of arbitrary SQL expressions on Arrow buffers using runtime code-generation in LLVM.

## What is LLVM?

> The LLVM Project is a collection of modular and reusable compiler and toolchain technologies. Despite its name, LLVM has little to do with traditional virtual machines. The name "LLVM" itself is not an acronym; it is the full name of the project.

It is a compiler system for C and C++.

> While LLVM is an alternative to GCC for general purpose compiling needs, it also provides Just-in-Time compilation capabilities that can incorporate runtime information to produce highly optimized assembly code for the fastest possible evaluation.

## What is an LRU cache?

There are many possible algorithms one can use for cacheing of data.  LRU stands for "Least Recently Used" and as the name implies, keeps track of what components were used when, and when it needs to discard items, discards the least recently used item first.

## What are hard and soft floating point maths?

"Hard" and "soft" refer to whether the computation is done on hardware or software.  

> Hard floats use an on-chip floating point unit. Soft floats emulate one in software. The difference is speed. 

## What is a MIME type/media type/IANA?
> A media type (also known as a Multipurpose Internet Mail Extensions or MIME type) is a standard that indicates the nature and format of a document, file, or assortment of bytes. 
> The Internet Assigned Numbers Authority (IANA) is responsible for all official MIME types, and you can find the most up-to-date and complete list at their Media Types page.

## What is Flight?

Flight is used for fast transfer of Arrow data via HTTP.

> [Flight is a] general-purpose client-server framework to simplify high performance transport of large datasets over network interfaces.

## What does "unsafe" mean in the context of Rust?

> Memory safety is the state of being protected from various software bugs and security vulnerabilities when dealing with memory access, such as buffer overflows and dangling pointers. For example, Java is said to be memory-safe because its runtime error detection checks array bounds and pointer dereferences. In contrast, C and C++ allow arbitrary pointer arithmetic with pointers implemented as direct memory addresses with no provision for bounds checking, and thus are potentially memory-unsafe.

> Rust is a multi-paradigm programming language designed for performance and safety, especially safe concurrency. Rust is syntactically similar to C++, but can guarantee memory safety by using a borrow checker to validate references. Rust achieves memory safety without garbage collection, and reference counting is optional.

> You can take five actions in unsafe Rust, called unsafe superpowers, that you can’t in safe Rust. Those superpowers include the ability to 1) dereference a raw pointer, 2) call an unsafe function or method, 3) access or modify a mutable static variable, 4) implement an unsafe trait, 5) access fields of unions.


# References

1. https://github.com/andygrove/datafusion
2. https://github.com/ballista-compute/ballista/blob/main/README.md
3. https://llvm.org/
4. https://www.dremio.com/announcing-gandiva-initiative-for-apache-arrow
5. https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)
6. https://en.wikipedia.org/wiki/Apache_ORC
7. https://orc.apache.org/docs/
8. https://stackoverflow.com/questions/3321468/whats-the-difference-between-hard-and-soft-floating-point-numbers
9. https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types
10. https://arrow.apache.org/blog/2019/10/13/introducing-arrow-flight/
11. https://en.wikipedia.org/wiki/Memory_safety
12. https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html
13. https://en.wikipedia.org/wiki/Rust_(programming_language)