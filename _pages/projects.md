---
layout: page
permalink: /projects/
title: projects
description:
nav: true
---


# **Powerlifted**: A Lifted Classical Planning System

[Powerlifted](https://github.com/abcorrea/powerlifted) is a classical planner
that plans directly over PDDL representations. In other words, it does not
*ground* the planning problems. Powerlifted is based on heuristic search, and it
has different search algorithms and proceedings implemented on top of it.

<br>

# A Collection of **Hard-to-Ground Domains**

The [htg-domains](https://github.com/abcorrea/htg-domains/) contains a
collection of *hard-to-ground* (HTG) classical planning domains. These domains
come from many different sources, which you can find described in the repo.

<br>

# **Grounders** for Planning

We also have tested [different grounders for planning
systems](https://github.com/abcorrea/asp-grounding-planning), although these do
not give you intermediate representations (e.g., SAS+) but simply output the
(relaxed) reachable actions and atoms. There is also a public implementation of
[Fast Downward using these different
grounders](https://github.com/abcorrea/downward-new-grounder).

Moreover, you might be interested in
[CounterAction](https://github.com/abcorrea/counteraction), our tool for
*counting actions without grounding*.
