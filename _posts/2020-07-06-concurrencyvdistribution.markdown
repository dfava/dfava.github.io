---
layout: post
title:  "Concurrency, Distribution, and the Go memory model"
date:   2020-07-07 13:00:00 +0100
categories: programming-languages
---
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$','$'], ['\\(','\\)']],
    processEscapes: true
  }
});
</script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

The Go memory model specifies two main ways in channels are used for synchronization:

1. A send onto a channel happens-before the corresponding receive from that channel completes.
2. The $k^{th}$ receive from a channel with capacity $C$ happens-before the $(k+C)^{th}$ send onto that channel completes.

Recall that *happens-before* is a mathematical relation, as discussed [here][mmhb].

Rule (1) above has been around for a while.  The rule is very similar to what was originally proposed by [Lamport in 1978][lamport78].  It establishes a happens-before relation between a sender and its corresponding receiver.
Rule (2) is a bit more esoteric.  It was not present in Lamport's study of distributed systems.  There is a good reason for that absence.

<!--more-->

> Go is a language in between concurrency and distribution.

Both concurrency and distribution speak of independent agents cooperating on a common task.  For that to happen, agents need to coordinate, to synchronize.  Although similar in many ways, *concurrency* and *distribution* are fundamentally different.  Because of this difference, synchronization in the setting of distribution differs from synchronization for concurrency.

In a concurrent systems, we assume that the agents are under/within a single environment.  In Go, for example, all agents (goroutines) are under a single umbrella, in this case the Go runtime.  This overarching environment allows us to assume that no messages are lost during transmission.

In distributed system, however, there is no such point of authority---at least not without making lots of extra assumptions about the system.  For example, it may be impossible to tell whether a message was received.  A network delay may be indistinguishable from a crashed/failed node.  This impossibility exist even if we label some node as the "authoritative source of information about the state of the system."  After all, what if we are unable to reach this special node?  In a distributed system, communication is no longer perfect, and we are forced to deal with this fact at some point.

Locks are often used to program concurrent systems, where the agents are located under a central resource manager.  This manager can be the operating system, or a language runtime with the help of the OS.  Different from locks, channels are a step towards synchronization in the setting of distribution.

Go borrowed rule (1) from Lamport's research on distribution.  On the other hand, rule (2) comes from the realization that Go is not all the way there.  Rule (2) allows for the use of channels as locks, with *send* acting as `acquire` and *receive* as `release` (see [previous post][channelsvlocks] for details):

```
  T0            T1
c <- 0     |  c <- 0
z := 42    |  z := 43
<- c       |  <- c
```

Rule (1) gives us an order, while rule (2) is related to mutual exclusion (an order exists, but we don't know which).  In a sense, rule (1) is [constructive or intuitionistic][intuitionistic], while rule (2) is [classical][classical].  If you are interested, you can find more on Section 3.5 of our paper [Ready, set, Go! Data-race detection and the Go language][readysetgo].


## Conclusion

While channels are typically used to program distributed systems, Go has a slightly different angle on message passing.  Go introduces rule (2), which takes into account the channels' capacity:

<ol start="2">
<li/>  The $k^{th}$ receive from a channel with capacity $C$ happens-before the $(k+C)^{th}$ send onto that channel completes.
</ol>

With rule (2), we can program channels as locks.  This puts the language in the spectrum between concurrency and distribution.



[mmp1]: /programming-languages/2020/03/05/memory-models.html
[mmp2]: /programming-languages/2020/03/06/weak-memory-models.html
[mmgo]: /programming-languages/2020/03/12/gomm.html
[mmhb]: /programming-languages/2020/03/11/mm-hb.html
[channelsvlocks]: /programming-languages/2020/06/24/channelsvlocks.html
[gomm]: https://golang.org/ref/mem
[lamport78]: https://dl.acm.org/doi/abs/10.1145/3335772.3335934
[readysetgo]: https://doi.org/10.1016/j.scico.2020.102473
[intuitionistic]: https://en.wikipedia.org/wiki/Intuitionistic_logic
[classical]: https://en.wikipedia.org/wiki/Classical_logic
