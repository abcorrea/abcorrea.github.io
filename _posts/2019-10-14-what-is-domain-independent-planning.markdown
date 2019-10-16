---
title: What is Domain-Independent Planning?
---

Let's say that you want to develop a solver for the [Rubik's cube
puzzle](https://en.wikipedia.org/wiki/Rubik%27s_Cube). Your solver receives as
input an initial configuration of the cube and it must find a sequence of
movements leading to a configuration where each face of the cube has a unique
color. There are several techniques that you can implement in your solver (e.g.,
heuristic search, group-theoretical methods, SAT formulations, etc.) but let's
leave it alone for now.

In essence, solving a Rubik's cube is a *classical planning problem*: you
receive an initial description of your world, a set of transformations that you
can apply to it, and a goal condition. The objective of your solver, or in the
context of planning your *planner*, is to find a sequence of these actions
transforming the initial configuration of the world into a new configuration
where the goal condition is satisfied. In planning terminology, the initial
description is called the *initial state*, the set of transformations are the
*actions*, and the goal description is (sometimes) simply called the *goal*. The
sequence of actions leading from the initial state to some state satisfying the
goal condition is a *plan*.

In our Rubik's cube example, the initial state is some arbitrary permutation of
the cubies, the actions are the rotations of the faces, and the goal is to find
a state where all faces have only cubies of a unique color.

But let's say that now want to develop a planner for another popular puzzle: the
[15-puzzle](https://en.wikipedia.org/wiki/15_puzzle). This is also a planning
problem: your initial state is an arbitrary configuration of the tiles, your
actions move adjacent tiles into the blank tile, and your goal is to rearrange
the tiles in numerical order such that the tile label "1" is at the top-left
corner and the missing tile is at the bottom-right corner. You can, once again,
develop a new specialized planner that receives an instance of the 8-puzzle as
input and outputs a plan for it. But is it really necessary to come up with a
new specialized planner for every new problem you want to solve? Could you maybe
use the same algorithm for both puzzles?

In fact, yes, you can. That is where *domain-independent planning* comes into
play. In this setting, we aim to find a single planner that can solve any
possible planning problem someone can come up with. Domain-independent planners
use some type of formalism to describe planning problems. These formalisms are
simply declarative languages where the user can specify the problem using a
pre-defined syntax. In this way, the planner must extract all knowledge of the
problem from its description, without having access to any prior information
about the problem. There are several formalisms that one can use (e.g., STRIPS,
SAS+, etc.), but let's abstract this for now. In general, any planning formalism
is simply a fragment of propositional or first-order logic that can represent
several different problems.

The main advantage of a domain-independent planners is that the user must only
care about *modeling* the problem and not about *how to solve* the problem. If
you have a problem you want to solve, you only need to describe it using some
specific formalism and let the planner do the hard part. However, not everything
is perfect. There is a clear trade-off between efficiency and
generalization. Naturally, a domain-independent knowledge has way less
information about the task it is currently solving and, as one might expect, it
usually performs worse than a specialized planner. This is the price one has to
pay to have an algorithm that solves not a single problem, but a whole family of
problems.

Planning has been one of the core ideas in AI for decades and there is a lot of
research activity in the area. There are several flavors of planning. The most
popular ones, besides classical, are probabilistic, partially-observable,
temporal, and FOND planning. The goal in any of these variants is the same: find
one planner, or algorithm, that can solve any possible planning problem.
Domain-independent planners are still mostly used in academia, but several
companies and specialized solvers obtained ideas and implementations from these
planners. Planning systems are very useful for problems in which the definition
is constantly changing. In these cases, you don't need to care about changing
your code but simply modify your formal problem definition. Another usage of
planning is when implementing your own solver would have high costs. Why put so
much effort and money developing a new solver when an off-the-shelf planner with
cutting-edge technology might be enough?

If you still have questions about what planning is and what it is used for,
consider reading the paper ["Everything You Always Wanted to Know About
*Planning* (But Were Afraid to
Ask)"](http://fai.cs.uni-saarland.de/hoffmann/papers/ki11.pdf) by Jörg Hoffmann.
