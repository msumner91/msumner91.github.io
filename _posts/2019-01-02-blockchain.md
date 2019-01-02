---
layout: post
title: Blockchain in F#
---

# Overview
This work was inspired by a curiosity for ML-like languages, the hype surrounding blockchains in 2018 and a similar post which used Python to the same end [1]. You can find the [source here](http://www.github.com/msumner91/blockchain-fs). The core of the idea is to build a basic web app that implements a blockchain in F# - here we make use of the Suave for the routing/http interface and Chiron for Json serde. 

One takeaway point is just how elegant and practical F# is as a language upon first glance...

1. Product/Sum types are elegant and concise as is the case in ML-like languages (and certainly more than Scala 2.x though Dotty will go some way to alleviate this)
2. Function args (rarely) need type annotations as the type system is simpler (e.g. no HKT) than Scala and uses the Hindley-Milner inference algorithm instead of local type inference.
3. Supports value types / structs for stack allocation and inlining.
4. Perhaps not a virtue of F# as much as CLR vs. the JVM but F# has support for proper tail calls. Because CLR supports tail opcode we can support proper tail calls in the general case (e.g. mutually recursive functions) and not just the self recursive case. In Scala/JVM land this is not possible without trampolines.

[1] https://hackernoon.com/learn-blockchains-by-building-one-117428612f46
