---
layout: post
title:  "Yesterday I defended the PhD"
date:   2021-06-15 13:00:00 +0100
categories: personal
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

As cliché as it sounds, the story of my PhD started in my childhood.
Growing up, I wanted to be a scientist. 
In my teenage years, I had my eyes on a [joint math and physics program][curso51]
at the *Universidade Estadual de Campinas* ([UNICAMP][unicamp]), Brazil.
I started the program in 2001 and still remember 
Professor Alcibíades Rigas class on linear algebra and analytical geometry.
Rigas used a graduate level book written in English as the main resource for this freshman level course---in a country where most don't speak English.
It's an understatement to say that the class was hard.
But rather than an exercise in gratuitous punishment,
Rigas helped us build a solid foundation.
I fell in love with the campus and the program, but I left midway through my freshman year.
While taking entrance exams in Brazil, I had also submitted applications for college in the US.
When notified of my acceptance at the Rochester Institute of Technology ([RIT][rit]), I chose the unknown over a life I had been falling in love with.
<!--more-->

Moving to the US was a momentous decision for me.
Leaving a liberal, public (free) university that is strong in theory and the sciences and going to a paid, conservative school with a focus on industrial application... had I made a big mistake?
The feeling of isolation, the cold, and the political climate post 9/11 weighed hard.  But I also made life-long friends during this time, and learned to embraced the engineer in me.
In the end, RIT did prepare us well for industry.
After college, I worked at Intel on one of the [first many-core CPU architectures][larrabee]. 
At Apple, I worked on the [first 64-bit cellphone processor to hit the market][a7].
But my childhood dream of being a scientist looked far away in the rear-view mirror.
So on the verge of becoming a father, with the encouragement and support of my wife, I took an order of magnitude pay-cut and made a u-turn into graduate school.


I enrolled in the PhD program in Computer Science at the University of California in Santa Cruz ([UCSC][ucsc]) in California.  Wanting to find my way back to math and science, I took classes in machine learning and in the theory of programming languages.
I became interested in logic and was exposed to formal methods.
But I struggled to find my footing,
and life in the US was not easy for two graduate students with a kid.
With the help of the [Good Country Index][goodcountry], I made a list of potential places to live. 
A serendipitous e-mail from [Olaf][] (now my PhD co-advisor) and the support from amazing friends put us in motion towards Norway.

At the University of Oslo ([UiO][uio]), I continued studying [programming languages and formal methods][psy].
In this thesis you may sense the pushes and pull of a person with mixed interests.
The operational semantics and the proof by simulation that appear early in the document come from wanting to deepen my mathematical background.
The work of manipulating symbols in a formal system, however, is more fitting to a theoretician than to the engineer who I had become.  So I am grateful to [Martin][martin], my advisor, for taking my interest and curiosity seriously, for encouraging me to develop my own research style, and for helping me bridge my knowledge gap.

I also wanted to build a modest trail, starting with real world source code and veering towards math. 
A trail that someone like my past self---a programmer who aspires to learn more but who does not yet have graduate-level training---might find useful.
With the goal of bringing the thesis' work to practice, I began looking at source code again.
My exposure to industrial code bases and my experience dealing its complexities helped me a lot.
I studied the thread sanitizer library ([TSan][tsan]), the [Go data race detector][goracedetector], and the implementation of channels in the Go runtime.
What started as tinkering developed into the latter part of the thesis.
In the process I was exposed to open-source development, which I have been interested in since my undergraduate studies.

I am tremendously grateful for the journey.  Risking opening *and finishing* with a cliché: I hope you will find the work interesting.  Thank you.

*That was the preface to my thesis.  Below is a technical blurb.  If you fond them to be interesting, you can check out the [PDF][thesis].*

<br/>

## Thesis blurb

Go is an open-source programming language developed at Google that has become the underpinning of large amounts of virtual infrastructure, especially in the area of cloud computing.  Inspired by Go, this thesis analyzes a programming environment where threads synchronize with each other via the exchange of messages over channels.  Go specifies this interaction on a document called the memory model, written in English.  We present a mathematical interpretation of the Go memory model document.  Our mathematization brings benefits.  For example, it allows us to more easily relate the language's design to the language's implementation.  As evidence, we were able to find and fix a non-trivial bug in the Go data-race detector.  Rooted in theory, our improvements to the Go data-race detector were incorporated into [release 1.16][go1dot16] of the language.  In this thesis ([PDF][thesis]), we share our experience applying formal methods to a large, real-world software project.


[thesis]: /thesis/dfava_thesis.pdf
[curso51]: http://www.upa.unicamp.br/cursao
[unicamp]: https://www.unicamp.br/unicamp/english/
[rit]: https://www.rit.edu/
[ucsc]: https://www.ucsc.edu/
[uio]: https://www.uio.no/english/
[psy]: https://www.mn.uio.no/ifi/english/research/groups/psy/
[martin]: https://martinsteffen.github.io/
[olaf]: https://www.mn.uio.no/ifi/english/people/aca/olaf/
[larrabee]: https://en.wikipedia.org/wiki/Larrabee_(microarchitecture)
[a7]: https://en.wikipedia.org/wiki/Apple_A7
[goodcountry]: https://index.goodcountry.org
[tsan]: https://clang.llvm.org/docs/ThreadSanitizer.html
[goracedetector]: https://golang.org/doc/articles/race_detector
[go1dot16]: https://golang.org/doc/go1.16#runtime
