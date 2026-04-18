---
title:  "AI and PR reviews"
categories: software-engineering
date:   2026-02-25 10:00:00 +0100
author: Daniel Fava
---

With AI doing so much coding, the bottleneck has become reviews.

- What do we do with those pull requests with thousand of lines of changes that the AI has created?
- Do we trust it and merge it?  Do we spend the time to review it?
- It might have taken 20 minutes to create the PR, but if it takes 2 days to review it, have we gained in efficiency?

There are many axis we can use to evaluate a change; for example:

- syntactical (typically easy to review, lower risk) versus semantical changes (typically harder to review, higher risk)
- changes in the critical path (e.g. movement of money) versus less critical (e.g. backoffice changes)
- reversible changes versus non-reversible ones (or reversible but incurring a large amount of clean-up effort)

In of all of these dimensions, there are subspaces where I'm totally fine with large changes driven by AI.

And there are some spaces where AI changes and AI reviewers should (IMHO) be driven and understood by a human.

My wish is that AI will help us undertake the huge yet boring refactorings that we keep of pushing into the future; like replacing libraries that are old and deprecated but that we don't dare to replace because the function signatures changed and the refactoring would touch thousand of lines in hundreds of files.

These are the types of changes are large (boring to be made by a human), but easy for AI and easy for a human to review.

What I really dislike is changes that are large and modify logic (as opposed to being syntactical).  Those types of changes (semantic changes) are hard to review and carry a lot of risk.  We need to try to break these down into paletable chunks that are digestible by a human.  I'm not quite ready to let an AI take care of banking accounting systems, for example. It's too risky; it would be a recipe for disaster.

<!--more-->

## Breaking it down

When adding a feature, or making a modification in general, we need to understand which parts fall in which subspaces, and we need to craft the work and the pull-requests into chunks.  Each chunk fitting into one of these subspaces.  Meaning.. don’t mix a risky semantical changes together with a thousand of lines syntactical change!

To me, a change is like a theorem you need to prove, and the implementation is the proof of the theorem.  Bare with me for a moment:

Modifications are typically like this:
- we want a lot of the system's behavior to stay the same.  Call it `C` for _constant_.
- we want some of the system's behavior to stop existing.  Call it `S` for _stop_.
- we want to add some new behavior into the system.  Call it `N` for _new_.

So, today the system behaves as `C union S`.
We want to make modifications so that the system behaves as `C union N`.  That means, continue doing `C`, stop doing `S` and add `N`.
Sure, sometimes we don't have `N`, and sometimes we don't have `S`, but in general we have both.

We want to go from `C union S` and modify it and we hope these modifications will create a system that satisfies `C union N`.
I'll make a analogy between pull-requests (PRs) and theorem proving... but first a quick reminder:

When proving something new, we start with a set of things we know, and we apply logical steps to the things we already know in order to grow the set of things we know.  More precisely:

- When proving something, we start with a set of assumptions. These are the things we know are true, or assume are true.
- To prove something we take small deduction steps.  Deduction rules are rules that were carefully crafted in order not to change the truth of a system; they can add truths we didn't know before, but they can't add a falsehood.
- So we start with the assumptions, cleverly apply deduction steps, and hope to arrive at a conclusion.

The analogy with theorem proving is the following:  

- Our set of assumption is `C union S`, meaning, it's how the system behaves today
- In order to get to `C union N`, we make small modifications starting from `C union S`.  Each modification is provably correct (we can look at the modification and see that it is correct).
- After a sequence of incontestable modifications, we arrive at `C union N`.

 `C union S` --PR1--> `C_1` --PR2--> `C_2` --> ... -- PRN --> `C union N`

Observations:

- A proof is the sequence of steps from a set of assumptions to a conclusion.
- Stating a conclusion is not the same as proving the conclusion.

Transferring that back to AI and pull requests:

- I can see that a modification to the system is correct when I reasonably understand the existing system (the assumptions) and I can see the steps that lead it to a new system (the sequence of small PRs in between).
- A very large PR that is not understandable is like stating a conclusion without giving any proof of the conclusion!

Trusting a large PR created by an AI without understanding it is like trusting a conclusion without understanding the assumptions and the reasoning behind it.
