---
layout: post
title: "Tech Lead, a retrospective"
date: 2026-04-18 00:00:00 +0000
---

I have been the tech lead of a ~12 person team for the past couple of years.
Recently I moved into the role of engineering manager.
Before I get too deep into the trenches of my new role, I wanted to capture what I've learned.

<!--more-->

## Gardener versus grand master chess player

Depending on the circle you are in, we hear about leaders as grand master chess players.  In this case, the game pieces are members of the team.  The media tends to gravitate towards the extraordinary, but in reality, the grand master metaphor is less applicable than it is made to be.

A more appropriate metaphor is that of a gardener.  Plants grow by themselves; the gardener doesn't _make_ plants grow.  But the gardener is instrumental in creating an environment that directs growth.
It’s about placing things where they can thrive, pruning when needed, and shaping the overall ecosystem.

With this metaphor in mind, here are some core principles that I've found helpful as a tech-lead.


## Hierarchy and optimization

When in doubt, optimize in this order:
 
1. company
2. team
3. individual

- If it’s slower for me (individual) but faster/better for the team, optimize for the team.
- If it’s slower for the team but faster/better for the company, optimize for the company.

I've touched on this on a previous post.  See [Late bloomer: Reflections on Becoming a Tech Lead][latebloomer].


## Creative boundaries, freedom and agency

There are good reasons to give people agency and freedom.  From a macro point of view, distribution can scale better than centralization.  And from a local point of view, intrinsic motivation is more powerful than extrinsic motivation.

But freedom does not necessarily mean:

* you get to re-write systems whenever you want
* you get to choose whatever programming language and technologies you want
* you get to make decisions that impact others without consulting them

So how do we balance freedom and agency against the need to organize and cooperate?

As a tech lead, I've come to think in terms of **creative boundaries**:

* you have freedom, and you give freedom within bounds
* most bounds are fluid; some are hard and non-negotiable
* you define the bounds as a team

It creates a shared contract with clear expectations, but loose enough to accommodate different personalities in the team.  This doesn't come for free; it takes effort. 
It means focusing on the _forming_ and _norming_ phases of a team's development so we can get through the _storming_ part and into _performing_ ([Tuckman's stages of group development](https://en.wikipedia.org/wiki/Tuckman%27s_stages_of_group_development)).

And to clarify, coordination doesn’t mean coordinating with the tech-lead.  We don’t want the tech lead to be a _single point of failure_.  Coordinating means individual contributors will think about involving stakeholders (whoever those are) when needed.

For example, before re-writing a large piece of code, have team members ask themselves:

* Who owns or uses this code, and have I talked to them?
* Who else should be aware or involved in this decision?
* Is this the right time to do this work?
* Does this align with what the team and organization need right now?


### Tight-loose-tight

When it comes to boundaries and expectations towards the team, a helpful model is the _“tight-loose-tight”_:

**Tight on what we want to achieve**

- Have a well-thought-out list of focus initiatives.  The responsibility for having such a list tends to lie with the product manager/owner.
The list usually combines external company initiatives and internal team needs.
It is put together iteratively, with input from the engineering manager and the tech lead.
The EM helps match people’s strengths and interests to the work. The tech lead brings input on complexity and effort.


**Loose on how to do it**

- Developers have a lot of freedom in defining the "how".

**Tight on measuring achievement**

- Follow-up on progress and end-result.  Note that achievement doesn’t need to be success; achievement means closure.  It's more about accountability (taking ownership and driving towards completion) than outcome (success).


## Decision making: It's about commitment, not consensus

**Focus on _commitment_ instead of _consensus_**.  In practice, that means not everybody in the team needs to agree; but everybody should be able to live with any decision.

**It is not a democracy**.  Taking a vote is not the only way to break out of interminable rounds of discussion.

You can propose a decision and ask people “can you live with this decision?"  It's not about making people happy.  It's also not about forcing a decision onto them.  It's about asking for their commitment.

Decision making is something I struggled with.  I shared technical leadership responsibilities with our area architect.  In retrospect, I suspect the team wanted more direction than what I gave them.  
I should have driven a clearer split of responsibilities between myself and our architect, and I should have invested in ways to settle our differences so that the team would benefit from a **consistent message**.


### The importance of having a consistent message

It is important for the team to get a relatively consistent message from tech-lead, manager, product manager, and architect.  To that end, we often discuss internally before presenting an initiative to the team.  This type of coordination is time well spent.

Even still, I struggled with the question: can I pull the “I’m the tech-lead” card in order to settle a discussion?  Would it be legitimate for me to claim the power of settling a discussion?
Even though some people wanted me to settle discussions, that authority was never formalized, so I didn’t treat it as part of my toolbelt.

Instead, I became comfortable asking probing questions.  And it is totally legitimate for a tech lead (or anyone in the team) to _ask questions and make observations_.  Through the questions I asked and the observations I made, I was often able to lead people and decisions towards an outcome without needing to pull the “tech-lead card” out of my pocket.


## Working outside of the team

The principles above mostly play out within the team. But the tech-lead’s role also extends outward.

Depending on your organization, this can be a significant part of the role—or barely visible.
While I was tech lead, our area architect was embedded in our team; so the role of communicating externally was shared between us. 
If that were not the case, I would have been expected to attend more sync meetings with architects and tech-leads of adjacent areas.

The tech-lead is also often responsible for security within the team, and interfacing with security initiatives coming from outside of the team.  You can delegate part of that, but you may still get pulled into discussions.


## Rituals

Our company worked in cycles: short planning periods (“flex”) followed by longer execution periods (“focus”).

### Weekly planning

During focus periods, we held a short weekly planning meeting between product manager, engineering manager, tech-lead, and architect.  We met on Friday to discuss what the next week might look like.

The planning wasn't driven by tasks from the backlog.  Instead, we shared our understanding of how the week went and where we thought we were in accomplishing our goals for the current _focus period_.


### Focus planning


The project manager, engineering manager, area architect, and tech-lead kept a finger on what was going on outside the team.  As the focus period ended, and we entered the _flex_ period, we would already have a sense of our standing towards the outside world; meaning, which company objectives would require our attention, which teams we would need to coordinate with, etc.  These ideas formed the basis for planning the next focus period.

Focus goals are crafted under the responsibility of the product manager.  The product manager, engineering manager, and tech-lead discussed the goals amongst themselves, and brought a fairly refined view of the world to the team.


## Examples

In addition to participating in planning, here are some concrete tasks I found myself doing regularly.

I kept an eye on communication channels to make sure questions being directed towards our team were being answered.  It wasn’t about answering them myself (or even knowing the answers).  It was just making sure someone from our team eventually addressed these questions.

I also kept an eye on pull requests for two reasons:
One, to take the opportunity to learn, teach, and share knowledge; thus nudging the code towards clarity and uniformity.  And two, to help keep the team focused.  It’s reasonable to ask: _what’s the context of this pull request, how does it fit into the weekly goals or focus period._  It is not about policing; it’s about understanding where people are coming from and making sure people are working together.

I worked on tasks that were likely to fall through the cracks.  I looked for tech debt that was in the foundations of what we do, and that was likely to remain unaddressed.  Those were the tasks that I gravitated towards.

During my time as tech lead, I paid attention to overall library design and refactoring; that involved:
* removing redundant code spread across libraries, and removing repeated type definitions
* seeking opportunities to define types so we benefit from type checking
* splitting packages that had grown too big and too confusing
* preventing creation of unwanted imports from one package into another
* adding missing features in underlying libs and unifying how we approach fundamental problems, like how to query the DB
* writing documentation, etc.


## Summary

Being a tech lead is a balancing act:

* If you **over-coordinate**, you risk becoming a bottleneck, and if you **under-coordinate**, you risk chaos and divergence
* If you **avoid conflicts**, you postpone decisions, but if you **overuse authority** you risk disengaging the team
* If you **ignore tech debt** you create long-term drag, but if you **focus only on coding** you miss the system view

The job is to create clarity with other leaders:
- crafting a consistent message,
- defining boundaries,
- planning initiatives,
- guiding decisions.

Clarity comes from repeatedly asking:

* what are we doing?
* why are we doing it?
* what matters now?
* what does “done” mean?


[latebloomer]: /leadership/2025/09/22/tech-lead.html
