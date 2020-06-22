---
layout: post
title:  "Weak memory models"
date:   2020-03-06 10:00:00 +0100
categories: programming-languages
---

In the [previous post][mmp1], we touched on consequences of our quest for performance.  By relaxing the order of execution of instructions, compilers are able to produce faster binaries, and processors are able to execute these binaries faster.  We want this rearranging to be done for us because efficient instruction scheduling is a science in itself.  Also, on the previous post, we touched on the concept of sequential consistency.  While sequential consistency imposes program-order across the board, relaxed memory models only preserve *single-thread semantics*.

In this post, we will be more direct about weak memory models.
<!--more-->


### Compositionality and single-threaded semantics

*Single-threaded semantics* means that each thread must not be able to tell that the order of its instructions has been messed with.  So, if I am a thread, I must not be able to tell that my instructions have been rearranged.  But if *you* are an external thread, then you may be able to tell the difference: you may be able to notice that *my* instructions are being executed out of program order.

What is interesting is that single-threaded semantics is not *compositional*:
You can make program transformations that preserve single-threaded semantics.  While each thread behaves the same before and after the transformations, the overall program behavior changes.  In other words, the whole is not always the same as the sum of its parts.

For example, let `x` and `y` be shared variables initialized to `0`, and `r1` and `r2` be registers local to the threads.

```
   T1               T2
x  = 1       |   y  = 1
r1 = y       |   r2 = x
print r1     |   print r2
```

You should convince yourself that this program can print the following pairs of values: `{(0,1), (1,0), (1,1)}`.

I will now swap the first two instructions in `T1` to obtain a new program.  Maybe this new program executes faster.

```
   T1               T2
r1 = y       |   y  = 1
x  = 1       |   r2 = x
print r1     |   print r2
```

The swap is not noticeable from the point of view of `T1`: before the swap, `T1` could print `0` or `1` and, after the swap, `T1` can still print `0` or `1`.  No change.  However, these are now the possible pairs of values printed by the program: `{(0,0), (0,1), (1,0), (1,1)}`.  What a minute!  The pair `(0,0)` is new.  This pair didn't exist before the swapping of `T1`'s instructions!  Is this allowed?!

In many relaxed memory models, the swap described above is totally acceptable.  Single-threaded semantics has not changed: individually, `T1` and `T2` behave exactly the same before and after the reordering of instructions.  However, it is possible to preserve single-threaded semantics and still obtain a different program as a whole.


## M for model and for meaning

There are different sources of weakness in a memory model.  Reordering of instructions can happens at the hardware level (at execution time) and at software level (at compilation time).  Also, in hardware, memory hierarchy (like processor store buffers, load buffers, caches) can lead to weaknesses in the memory model.  In fact, a bewildering number of models exist.


How can I then tell how my program really behaves?!  One answer is, a compiler makes sure a program behaves the same across different architectures.  The compiler does so by inserting the proper synchronization primitives for the given *compilation target*.
When pondering about program behavior and synchronization, however, it would be unreasonable to expect a programmer to peek into the compiler.
Luckily we don't have to do that.  Languages come with a *memory model*: a document that serves as a contract between the programmer and the compiler.

Note that the exists a tension here.  Programmers expect clear constructs and simple explanations.  Compiler writers want to implement complex optimizations and want freedom to evolve a language.   These two camps can find themselves at odds.  It is the job of a language's memory model to bring these camps together.  Regardless of the complexity of the optimizations, a program shall behave as described by the model.

Defining a reasonable memory model is juggling act.  We will look at a real-world memory model soon.  But first, in the [next post][mmhb] we cover concepts used in the specification these models.


[mmp1]: /programming-languages/2020/03/05/memory-models.html
[mmhb]: /programming-languages/2020/03/11/mm-hb.html
[gomm]: https://golang.org/ref/mem
[goruntime]: https://golang.org/pkg/runtime
[lamport79]: https://dl.acm.org/doi/abs/10.1145/3335772.3335935
[adve95]: https://ieeexplore.ieee.org/abstract/document/546611

