---
title: This Week on the Arrow Dev Mailing List
author: ''
date: '2021-04-27'
slug: this-week-on-the-arrow-dev-mailing-list
categories: []
tags:
  - arrow
---

Inspired by ["This Week in Ballista"](https://ballistacompute.org/this-week-in-ballista/), and my desire to keep up-to-date with everything happening in Arrow, here's my first "This Week in Arrow", summarising the main occurrences on the dev mailing list from 20th - 27th April 2021.

## Release candidate 1

There were votes on a few release candidates.  RC1 was blocked due to a number of issues.  There was an update which changed the default memory pool to prefer mimalloc instead of jemalloc on MacOs, as this had been [shown to lead to better peformance on macOS](https://issues.apache.org/jira/browse/ARROW-12316).  This code change [hadn't fully achieved this](https://issues.apache.org/jira/browse/ARROW-12485) and so a [further change](https://github.com/apache/arrow/pull/10117/files) was made to add this update.

There was a bug in a new API whereby if there were errors during scanning, [`ScanBatches()` hung instead of erroring](https://issues.apache.org/jira/browse/ARROW-12487).  Code changes were [added](https://github.com/apache/arrow/pull/10115/files) to ensure that errors were raised.  

## Gandiva LRU cache replacement

There was a discussion around replacing the LRU cache in Gandiva.  Whereas LRU (least recently used) discards things based on last usage, the proposed new cache algorithm also takes into account the building time for different expressions.  

## Copying Rust to new repos

The Rust components are being copied across to the new repositories and there was discussion around filtering git history via git-filter-repo, updating the CI, adding integration tests, and getting ready to accept PRs in the new repos.

## File extension

There was a discussion around registering the Arrow format with IANA as a media type.  There were 2 different types being discuss; streaming and the file itself, and the general opinion appreared to be that they should be different types.

## Random number generation
Random number generation was slow on ARM64, with significant differences between different compilers (with clang being much slower than gcc).  It was caused by soft-float math, and resolved by supplying the "-ffast-math" argument to clang.

## Rest parquet2

One contributor has been experimenting with re-writing the Rust parquet implementation which doesn't use "unsafe" (keyword used to allow functionality that doesn't guarantee memory safety), improving performance, and other things.  There is discussion of moving their implementation to an official Apache repo and people suggest how to go about it.

## Python-datafusion
Discussion around adding python-datafusion into the project.  Python-datafusion allows the use of Datafusion from Python.  Questions are asked around whether the plan is to move it into the monorepo or kept as a separate apache repo, and the author suggests at least doing releases separately so it can have independent versioning from pyarrow and not automatically bundle it with pyarrow.

## compute::isin rejects duplicates

One of the C++ compute functions, `isin` has parameter `value_set`, which raises an error if it contains duplicate values, in arrow 4.0.0.  A user notes that this behaviour is different in Arrow 2.0.0 and a ticket is opened to change this behaviour.

## pyarrow custom metadata

A user asks a question about the user of custom metadata in PyArrow - this is a feature in pandas that is not fully implemented in PyArrow.  There is also clarification that there are not plans to migrate all features from pandas to pyarrow, and further clarification that pyarrow is intended to be a back-end whereas pandas is both back-end and front-end.  A ticket was opened around adding examples of using custom metadata.

## Release candidate 3

The release candidate was verified by various people in multiple formats on multiple platforms, the vote was carried, and a draft blog post about the release was started.

## Miscellaneous

- A userguide was added for DataFusion and Ballista
- There was a Rust sync call  
- Discussion in the Go package around functionality to write to JSON and clarification that the internal package was related to integration tests and not aimed at end users, though there was alternative functionality elsewhere that met the OP's needs.
- There was a request for functionality that enabled app metadata from a Flight stream in Go; this is currently implemented in Java, and a ticket was opened to add a Go implementation too.
- A user struggled to find the JS documentation and was directed to its various sources
- Discussion of the Arrow JS meetup
- User asking about whether interoperability is expected between Go and Rust API and clarification that it is
- User asking about whether a particular data type (dense union) is supported in parquet, and clarification that it is not.
- A discussion that turned into a bug report around NumPy buffers setting different attributes around things being mutable 
- Mention of a potential bug to do with Pyarrow RecordBatchStreamWriter and dictionaries
- Discussion around the Julia migration - the plan is to follow the same process as the Rust one, and now the Rust migration is complete, the discussion is reopened.