---
layout: post
title:  "Channels vs Locks"
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

With channels, we typically establish that two things are synchronized because A happens-before B.  We often know the order in which they happen.  In this post, we'll see a use (or "misuse") of channels.  We will be able to establish that two things are synchronized, but we won't know the order between them.  We won't know which happened first.  We will then relate and contrast channels with locks: how are they different, how are they similar.

<!--more-->

The typical example of channel communication is:

```
  T0            T1
z := 42    |  
c <- 0     |  <- c
           |  z := 43
```

Thread `T0` does something (write to the shared variable `z`) and informs a partnet, `T1` by sending on a channel.  `T1` learns about the write to `z` by receiving from the channel.  `T1` can then use `z` without causing a data race.

The Go memory model specifies two main ways to synchronize with channels:

1. A send onto a channel happens-before the corresponding receive from that channel completes.
2. The $k^{th}$ receive from a channel with capacity $C$ happens-before the $(k+C)^{th}$ send onto that channel completes.

Rule (1) is the rule used to reason about the example above: `T0`'s send happens-before `T1`'s receive.  Rule (2) is a bit more esoteric.  It allows us to use (or "misuse", depending on your point of view) channels as locks.  

## Channels as locks

Here is an example of channels being used as locks.


```
  T0            T1
c <- 0     |  c <- 0
z := 42    |  z := 43
<- c       |  <- c
```

In this example, the channel `c` has capacity 1.  Threads `T0` and `T1` are both trying to access some shared resource, say `z`.  Before accessing `z`, a thread sends a message on the channel `c`, and receives from the channel afterwards.


Note that the send and its corresponding receive do not contribute to synchronization in this example.  The send is matched by a receive from the same thread; nothing new is learned from this exchange.  Rule (1) is mute here: the receive is in happens-before the send not just because of rule (1) but, more obviously, because of program order.  Yet, this program is properly synchronized.

This program is properly synchronized because the channel is acting like a lock: *send* is acting like the `acquire` operation, and the *receive* as the `release`.  What allows for this interaction is Rule (2).
To see that, let us assume that `T1` is the first thread to perform the send operation.  (We could easily apply the same logic assuming `T0` was first.)  Since the channel has capacity 1, `T0` will not be able to send onto the channel until `T1` has received from it.  Rule (2) then links the reception by `T1` to the send by `T0`:  the 0<sup>th</sup> receive happens-before the 1<sup>st</sup> send completes.  By this reasoning, we know that the write `z := 43` by `T1` in the past of `T0`.  Therefore, `T0` can access `z` without causing a data race.

For a rigorous discussion, see Section 3.5 of our paper [Ready, set, Go! Data-race detection and the Go language][readysetgo].


## Conclusion

Go channels are a bit more than channels in their pure sense.  In particular, rule (2), which allows us to use channels as locks, gives us extra power.  In the [next post][concurrencyvdistribution] I argue that this power is not necessarily a good thing.  Spoiler alert.  The possibility of using channels as locks is a good thing when it comes to concurrency.  However, this power makes Go less of a language for distribution.

Also, although we can make channels behave as locks, in [this post][everywordcounts] I discuss how synchronization through locks is *fundamentally different* from synchronization via channels.  The neat thing about the post is that we'll get to explore the Go runtime.  We'll also be bring together many of the concepts we've talked about in this blog so far.


[mmp1]: /programming-languages/2020/03/05/memory-models.html
[mmp2]: /programming-languages/2020/03/06/weak-memory-models.html
[mmgo]: /programming-languages/2020/03/12/gomm.html
[mmhb]: /programming-languages/2020/03/11/mm-hb.html
[concurrencyvdistribution]: /programming-languages/2020/07/07/concurrencyvdistribution.html
[everywordcounts]: /programming-languages/2020/07/08/everywordcounts.html
[gomm]: https://golang.org/ref/mem
[lamport78]: https://dl.acm.org/doi/abs/10.1145/3335772.3335934
[readysetgo]: https://doi.org/10.1016/j.scico.2020.102473
