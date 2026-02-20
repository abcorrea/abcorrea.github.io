---
layout: page
permalink: /projects/
title: projects
description:
nav: true
---

# **LLM-Generated Heuristics** for Classical Planning

We use LLMs to generate Python heuristic functions for classical planning,
studying how to integrate LLM-generated code into planning systems while
preserving formal correctness guarantees. Published at **NeurIPS 2025**. Code
available on [Zenodo](https://zenodo.org/records/17400964).

<br>

# **Powerlifted**: A Lifted Classical Planning System

[Powerlifted](https://github.com/abcorrea/powerlifted) is a classical planner
that plans directly over PDDL representations. In other words, it does not
*ground* the planning problems. Powerlifted is based on heuristic search, and it
has different search algorithms and proceedings implemented on top of it.

<br>

# **Grounding** Infrastructure for Planning

- [htg-domains](https://github.com/abcorrea/htg-domains/): A collection of
  *hard-to-ground* (HTG) classical planning domains from many different sources.
- [Grounders for planning](https://github.com/abcorrea/asp-grounding-planning):
  Different grounders that output (relaxed) reachable actions and atoms, with a
  [Fast Downward integration](https://github.com/abcorrea/downward-new-grounder).
- [CounterAction](https://github.com/abcorrea/counteraction): A tool for
  *counting actions without grounding*.
