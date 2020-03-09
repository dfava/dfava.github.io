---
layout: post
title:  "Memory models, part I"
date:   2020-03-05 10:00:00 +0100
categories: programming-languages
---
In this series of posts we will visit the concept of *memory models* using "real-world" examples.  We will look behind the scenes and into a programming language's implementation.  We will see how the specification of the memory model relates to the language's implementation.  This journey will take a few posts.

Below we cover some basics:  what is a memory model and why do these models matter.  We will touch on concepts associated with multi-threading and synchronization, such as the concept of *sequential consistency*, *weak-* or *relaxed-memory*, *atomicity*, etc.  In future posts we will discuss the [Golang memory model][gomm] and we will look into [Go runtime][goruntime] source code.  But first, a confession.

My background is in engineering, not in computer science, and I managed to have a fine career in Silicon Valley without having to think too hard about memory models.  For the better part of my tenure, I wasn't even aware of the concept.  Turns out memory models are quite interesting.  And it amazes me how anything works given how complex and elusive memory can be.  We will have plenty to talk about, even from the smallest examples.  You may be surprised by how much we can unpack from just six lines of code or so.

Let's start with this example here.  What does this program do?

```
     T1         |    T2
z    = 42       |   if (done)
done = true     |     print(z)
```

There are two threads, `T1` and `T2`, and they are trying to synchronize with each other.  `T1` does some work, represented here by the setting a shared variable `z` to `42`.  `T2` checks if the work is done, and if so, it prints the value of the shared variable out.

The answer to "what does this program do" is: it depends.
`T2` may execute before `T1`, in which case `done` is false and nothing is printed.

But even if `T2` observes the value of `done` to do `true` it doesn't mean it will print the value of `42`.  It is possible for `T2` to print zero.  If you are not so sure about why that's the case, then read on...

## A short primer on memory models

The question of "what does the above program do?" is settled by a memory model.  So what exactly is a memory model?  Let us look at what people have to say.  Here is one way to describe it:

> [A memory model is] a formal specification of how the memory system will appear to the programmer, eliminating the gap between the behavior expected by the programmer and the actual behavior supported by a system.

The quote comes from a [famous and very clarifying research paper][adve95].  But you probably don't get a clear picture given this definition.  So, let's try again:

> A memory model dictates what values a **read** from memory can return given previous **writes** to memory.

Wait a minute... if I read before writing, I should read some uninitialized or pre-initialized value.  And if I read after writing, then I read what I wrote.  What's the catch?  The catch is that the definition seems too trivial: it doesn't talk about what's actually interesting about memory models.  The definition does not touch on *concurrency*.

If I have one agent reading and writing to memory, then a memory model is trivial: we read what we wrote.  But what if many agents are writing to memory.  What values can we observe then?  That's the question a memory model helps you answer.

Here is another definition, this time from the [Go programming language][gomm]:

> The Go memory model specifies the conditions under which reads of a variable in one goroutine can be guaranteed to observe values produced by writes to the same variable in a different goroutine.

This is specific to Go, but it doesn't have to be.  We could just as easily replace "The Go memory model" by "A memory model", and "goroutine" by "thread":

> A memory model specifies the conditions under which reads of a variable in one thread can be guaranteed to observe values produced by writes to the same variable in a different thread.

There we go.  That's a fine definition.  It talks about reads observing writes and talks about what makes a memory model interesting: multiple agents interacting via shared memory.

## The problem with memory models

In computing, we often encounter a trade-off between performance and complexity, and it generally looks like this:

![performance vs complexity](/img/perf_comp_graph_sc_wm.png)

We want performance, but performance comes at a cost: increased complexity.  The same trade-off happens with memory models.  There isn't one absolute memory model but many.  At the bottom left we have *sequential consistency* (SC).  Memory models that are sequentially consistent are easier to understand, but don't yield very good performance.  As we move to the right, we move towards what is called *weak memory* (WM).  There are many flavors of weak memory models.  These models differ based on the specifics of how they trade simplicity for performance.

But before we get ahead of ourselves, let us see what a sequentially consistent memory model looks like.  The [original definition from 1979 by Lamport][lamport79] says:

> [A memory model is sequentially consistent when] the results of any execution is the same as if the operations of all the processors were executed in some sequential order, and the operations of each individual processor appear in this sequence in the order specified by its program.

Say what?!

Let us try again with another definition:

> In a sequentially consistent model, memory acts as a shared global repository where operations appear atomic and in program order.

There are two concepts here: *atomicity* and *program order*.

*Atomicity* tells you that if two people try to write to the same location at the same time, the result will not be a mumble jumble with part of one write and part of another.  Instead, one write will "win" by overwriting the other.

*Program order* is the order of instructions in the program text.  The compiler is allowed some freedom rearranging instructions, and processors are also allowed some freedom in the order of execution of these instructions.  However, when it comes sequentially consistent memory models, the rules are pretty strict.  Instruction reordering can still happens.  Sequential consistency says that the result must be *as if* instructions were executed in program order.  The compiler or the hardware can still compile/execute instructions out-of-order, as long as we are not able to tell the difference.

Enough with definitions for now.  Let us go back to our original example.  I am going to label the program locations `(A)`, `(B)`, `(C)`, and `(D)`.

```
     T1            |    T2
z    = 42     (A)  |   if (done)     (C)
done = true   (B)  |     print(z)    (D)
```

In a sequentially consistent world, if `T2` observes the value of `done` to be `true`, then it must be because `T1` set it to true in `(B)`.  Because instructions must appear to have been executed in program order, then it must be that `T1` has already set `z` to `42` in `(A)`.  Therefore, the print statement in `(D)` will necessarily print `42`.  Bingo!

Life is good and we can go home now.  Well... except that most programming languages and processors are not sequentially consistent.  In our ever quest for performance, compilers and processors deviate from sequential consistency.  These deviations are also called *relaxations*, as they *relax* the order of instructions and their execution.

## *Weak* or *relaxed* memory models

By relaxing the order of execution of instructions, compilers are able to produce faster binaries, and processors are able to execute these binaries faster.  While sequential consistency imposes program-order across the board, relaxed memory models only preserve *single thread semantics*.

In other words, each thread must not be able to tell that the order of its instructions was messed with by the compiler or hardware.  So, if I am a thread, I must not be able to tell that my instructions have been messed with.  But if *you* are an external thread, then you will be able to tell the difference: you will be able to notice that *my* instructions are being executed out of program order.

Here is a concrete example.  The `x` and `y` are shared variables initialized to `0`.  The `r1` and `r2` registers local to the threads.

```
   T1               T2
x  = 1       |   y  = 1
r1 = y       |   r2 = x
print r1     |   print r2
```

You should convince yourself that the program can print the following pairs of values: `{(0,1), (1,0), (1,1)}`.

I will now swap the first two instructions in `T1` to obtain a new program.  Maybe this new program executes faster.

```
   T1               T2
r1 = y       |   y  = 1
x  = 1       |   r2 = x
print r1     |   print r2
```

The swap is not noticeable from the point of view of `T1`: before the swap, `T1` could print `0` or `1` and, after the swap, `T1` can still print `0` or `1`.  No change.  However, these are now the possible pairs of values printed by the program: `{(0,0), (0,1), (1,0), (1,1)}`.  What a minute!  The pair `(0,0)` is new.  This pair didn't exist before the swapping of `T1`'s instructions!  Is this allowed?!  In many relaxed memory models, this is totally acceptable.  Single threaded semantics has not changed: individually, `T1` and `T2` behave exactly the same before and after the reordering of instructions.  However, it is possible to preserve single threaded semantics and still obtain a different program as a whole.  Whole program semantics is not *compositional*.

There are different sources of weakness in a memory model.  Reordering of instructions can happens at the hardware level (at execution time) and at software level (at compilation time).  Also, in hardware, memory hierarchy (like processor store buffers, load buffers, caches) can lead to weaknesses in the memory model.  In fact, a bewildering number of models exist.

How can I then tell how my program will behave when it gets compiled to different hardware architectures?!  That's the job of a programming language's memory model.  Many programming languages specify a memory model so that you don't have to worry about this question.  You program according to the language's memory model and compiler makes sure it behaves the same across architectures by, for example, inserting the proper synchronization primitives for the given *compilation target*.

[gomm]: https://golang.org/ref/mem
[goruntime]: https://golang.org/pkg/runtime
[lamport79]: https://dl.acm.org/doi/abs/10.1145/3335772.3335935
[adve95]: https://ieeexplore.ieee.org/abstract/document/546611

## Closing the loop

Going back to the original example and the question of how it behaves.
Under weak memory, it is possible for the instructions in `T1` to be swapped, for example, if they are deemed to execute faster that way.

```
     T1         |    T2
done = true     |   if (done)
z    = 42       |     print(z)
```

The single threaded behavior of `T1` is still the same: both `done` and `z` get set to `true` and `42`.  However, the swap breaks the intent of the overall program.  It is now possible for `T2` to perceive `done` being `true` and for the print-statement to then print the uninitialized value of z.
