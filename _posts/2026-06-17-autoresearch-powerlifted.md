---
layout: post
title: Autoresearch to Optimize Powerlifted
date: 2026-06-17 12:00:00 +0100
description: An Autoresearch loop improved the performance of Powerlifted in 12 hours.
tags: [planning, llms, experiments]
categories: [agentic-coding]
---


All the cool kids these days try to use "*Autoresearch*" loops to let their
coding agents run wild with their softwares. [Jendrik
Seipp](https://mrlab.ai/jendrik-seipp/) has successfully tried it out in some of
his projects, and he borrowed me his handcrafted agentic skill to try on my own.

My target was [Powerlifted](https://github.com/abcorrea/powerlifted). The rule
was simple: make search faster *without changing what it does*. I used Claude
Code with Opus 4.8. The entire setup and logs are [available
online](https://github.com/abcorrea/powerlifted/tree/autoresearch/search-time-2026-06-07). This
setup is entirely credited to Jendrik. The initial prompt was the following:

```md
Run autoresearch on this repo. The setup already exists — do not recreate
it.

1. Read `autoresearch.md` (the brain file), the tail of
   `autoresearch.jsonl`, and `git log --oneline -20`, then continue from
   wherever the loop left off.
2. Loop forever: think → edit → `./autoresearch.sh` → decide with
   `.claude/skills/autoresearch/scripts/decide.py` → commit or revert →
   append one line to `autoresearch.jsonl` → repeat. Never stop to ask
   whether to continue; I will interrupt when I want to steer.
3. Hard rules, non-negotiable:
   - Behavior-preserving changes only. The harness enforces this via
     `benchmarks/reference.json`; never regenerate or edit that file, the
     suite, the harness scripts, or anything listed as off-limits in
     `autoresearch.md`.
   - Keep a change only when `decide.py` says KEEP; revert on DISCARD,
     crash, or a `./autoresearch.checks.sh` failure, even if the numbers
     looked good.
   - Strictly one planner execution at a time, and nothing else CPU-heavy
     while sweeps are being timed.
   - Never use `git clean`. Revert with `git checkout -- .` and remove new
     files by hand.
4. Update the **What's Been Tried** section of `autoresearch.md` every 5–10
   experiments so a future agent does not repeat dead ends.
   ```

There is a brain file (`autoresearch.md`) that Claude reads and rewrites as it
goes; a script that rebuilds the planner, runs a sanity check, and then
benchmarks three runs over a fixed suite of 34 (config, task) pairs. The
reference numbers are stored in `reference.json` where the we record, for every
pair, the exit code, plan cost, expansions, and generations expected. Every
sweep is checked against it, and any mismatch fails the run and reverts the
change. On top of that, a small decision script compares the candidate's timings
against the current best and returns `KEEP`, `DISCARD`, or `RERUN`. Claude logs
each experiment as one line in a ledger, commits the `KEEP`s, reverts the rest,
and never stops to ask permission.

I left it running for ~12 hours. I had to prompt it to continue twice, and ask
it to correct a minor issue once (more on that below).  But what did it actually
do?  Most of the experiments were discards, which is what I expected. Over about
a dozen experiments it kept four, which fall into three ideas:

1. It cut overhead in the grounder used by the delete-relaxation heuristics: a
   flat vector with a single scan instead of a hash map with a double lookup,
   reused buffers, hash tables cleared instead of reallocated.
2. It removed redundant work in the successor generator. For example, once a
   join has enforced a static inequality precondition, every later join
   preserves it, so re-checking it after each join is useless. It now tracks
   which preconditions are already satisfied and skips the rechecks.
3. It replaced small `std::vector`s with small-buffer-optimized vectors that
   keep their elements inline. (This is the [`small_vector` from
   Boost,](https://www.boost.org/doc/libs/1_85_0/doc/html/boost/container/small_vector.html).)
   The same idea applied twice: once in the weighted grounder, and once in the
   successor generator. These two were the biggest wins by far.

(The small-vector work first used for `boost::container::small_vector`. But
Boost is [no longer a Powerlifted dependency since a few
months](https://github.com/abcorrea/powerlifted/commit/3ad8a8148bf25fda862416e9477666a375c90cf5),
and I do not want it to become one again for a 200-line container. So I
intervened and told Claude explicitly to not do so --- and then I updated the
prompt above to always mention that. Claude implemented a small, self-contained
`small_vector` instead.)

All of it was evaluated on a micro-suite on my laptop, so I ran a proper
experiment with [lab](https://lab.readthedocs.io/en/stable/): a combination of
IPC domains from the
[downward-benchmarks](https://github.com/aibasel/downward-benchmarks/) and the
[hard-to-ground](github.com/abcorrea/htg-domains/) suites. I used 1800 seconds
and 16 GiB per task, which I always did when evaluating Powerlifted for papers,
and compared the final commit against the baseline planner.

The results are... quite good (gasp!). They are [available online](https://abcorrea.github.io/assets/tmp/autoresearch-powerlifted.html). On the instances solved by both (baseline
and autoresearch version), search time dropped by 1.72x for GBFS with FF, for
example. Coverage improved as a side effect in all tested configurations as
well. No previously solved instance was lost.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/autoresearch-search-time.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/autoresearch-peak-memory.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    One configuration (<code>alt-bfws1</code>) as a representative example: per-task search time (left) and peak memory (right), with the baseline on the x-axis and the optimized branch on the y-axis. Search time sits below the diagonal (faster); peak memory sits a touch above it (the small price of inlining). The full numbers are in the <a href="https://abcorrea.github.io/assets/tmp/autoresearch-powerlifted.html">lab report</a>.
</div>

So are these changes actually brand new? Yes and no. At the end, this *autoengineering* is not research. None of the changes is a completely new idea. For example, [Powerlifted has an open-issue since 6 years to optimize (in)equality checks](https://github.com/abcorrea/powerlifted/issues/23). But I never judged it worth doing. Luckily, Claude thought otherwise. Similarly, the small vector optimization is a known C++ optimization, and [we even have a comment on it on the Powerlifted source code](https://github.com/abcorrea/powerlifted/blob/af421dbfdb5514a2f1fb8db07ac5349481e7c25b/src/search/novelty/achieved_ground_atoms.h#L11). But, interestingly enough, both optimizations were applied in non-obvious ways: the inequality precondition issue had no concrete details to be implemented, ansd the small vector optimization was applied to a *different* part of the code than we had initially considered.

Initially, I thought that Claude would conjure up lots of micro-optimizations that would sum up to a decent improvement. At the end, at least I have implementations of optimizations that I already wanted to do myself anyway. Everyone wins.
