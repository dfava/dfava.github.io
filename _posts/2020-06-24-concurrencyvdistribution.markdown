---
layout: post
title:  "Concurrency, Distribution, and the Go memory model"
date:   2020-06-24 13:00:00 +0100
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

Go borrowed rule (1) from Lamport's research on distribution.  On the other hand, rule (2) comes from the realization that Go is not all the way there.  Rather, Go is a language for concurrency.  As we will see next, it is because of rule (2) that we can use a channel as a lock.



## Channels as locks

Rule (2) allows us to use (or "misuse", depending on your point of view) channels as locks.  Here is an example.  The channel `c` below has capacity 1.  Threads `T0` and `T1` are both trying to access some shared resource, say `z`.  Before accessing `z`, a thread sends a message on the channel `c`, and receives from the channel afterwards.




```
  T0            T1
c <- 0     |  c <- 0         # Send the value of 0 onto channel c
z := 42    |  z := 43
<- c       |  <- c           # Receive a value from channel c
```

Note the threads do not exchange messages.  In this example, a *send* and its corresponding *receive* do not contribute to synchronization!  The send is matched by a receive from the same thread---nothing new is learned from this exchange.  Yet, this program is properly synchronized.  

Note also that rule (1) is mute here.  Rule (1) is obviated by *program order*.  Yes, the *send* is in happens-before to the corresponding receive.  But this happens-before relation can be derived simply because the *send* appears before the *receive* in the program text.  

How is this program synchronized then?  The program is properly synchronized because the channel is acting like a lock: the *send* is acting like `acquire`, and the *receive* as `release`.  Rule (2) is what allows for this interpretation.

To really convince ourselves that the program is synchronized, let us assume that `T1` is the first thread to perform the send operation.  (We could easily apply the same logic assuming `T0` was first.)  Since the channel has capacity 1, `T0` will not be able to send onto the channel until `T1` has received from it.  Rule (2) then links the reception by `T1` to the send by `T0`:  the 0<sup>th</sup> receive happens-before the 1<sup>st</sup> send completes.  Therefore, the write `z := 43` by `T1` in the past of `T0`.  Finally, we conclude that, `T0` can access `z` without causing a data race.

## Conclusion

We placed Go in a spectrum between concurrency and distribution.  We saw how rules  from the Go memory model are used for synchronization---the rules have different flavors: rule (1) gives us an order, while rule (2) is related to mutual exclusion (an order exists, but we don't know which).  In a sense, rule (1) is [constructive or intuitionistic][intuitionistic], while rule (2) is [classical][classical].  If you are interested, you can find more on Section 3.5 of our paper [Ready, set, Go! Data-race detection and the Go language][readysetgo].

Even though we can make channels behave as locks, I will argue that synchronization through locks is *fundamentally different* from synchronization via channels.  That's the topic of the next post. <!-- TODO [next post][channelsvlocks]. -->


[mmp1]: /programming-languages/2020/03/05/memory-models.html
[mmp2]: /programming-languages/2020/03/06/weak-memory-models.html
[mmgo]: /programming-languages/2020/03/12/gomm.html
[mmhb]: /programming-languages/2020/03/11/mm-hb.html
[channelsvlocks]: TODO
[gomm]: https://golang.org/ref/mem
[lamport78]: https://dl.acm.org/doi/abs/10.1145/3335772.3335934
[readysetgo]: https://doi.org/10.1016/j.scico.2020.102473
[intuitionistic]: https://en.wikipedia.org/wiki/Intuitionistic_logic
[classical]: https://en.wikipedia.org/wiki/Classical_logic
