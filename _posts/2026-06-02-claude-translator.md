---
layout: post
title: Translating Fast Downward's Translator to C++ Using Claude Code
date: 2026-06-02 12:00:00 +0100
description: We translated Fast Downward's translator from Python to C++ using Claude Code.
tags: [planning, llms, experiments]
categories: [agentic-coding]
---


There was a lot of talk about the recent [Bun rewrite in
Rust](https://github.com/oven-sh/bun/pull/30412). Anthropic [acquired
Bun](https://www.anthropic.com/news/anthropic-acquires-bun-as-claude-code-reaches-usd1b-milestone)
at the end of last year, and recently announced that the rewrite from Zig to
Rust was [done using their new "dynamic workflow" orchestration
feature](https://claude.com/blog/introducing-dynamic-workflows-in-claude-code).

This made me curious to see how Claude Code would do porting [Fast
Downward](https://www.fast-downward.org/latest/)'s translator from Python to C++. The implementation is not trivial, and the [algorithms are quite sophisticated](https://www.sciencedirect.com/science/article/pii/S0004370208001926). I have actually [ported the grounder component to C++](https://github.com/abcorrea/grounder) during my Ph.D., so I had *some* experience with the same task. The code is [available online](https://github.com/abcorrea/downward-claude-translator).

I forked Fast Downward and prompted Claude Code (with Opus 4.7 and high thinking effort) with the following instructions:

```md
You are an expert systems programmer specialized in C++ and automated planning.

Your objective is to translate the Python "translator" component (@src/translate) of the Fast Downward planning system into high-performance, modern C++ (C++20 or later). The core goal of this translation is to eliminate Python overhead, optimize memory layouts, and speed up the PDDL-to-SAS+ translation phase.

The generated C++ code must match the styling, conventions, and architectural philosophy of Fast Downward's existing C++ search component (@src/search).

## Testing and Evaluation Constraints
When validating, testing, and benchmarking your C++ implementation, you must adhere to the following strict execution environment limits:
* **Time Limit:** Exactly 120 seconds maximum per run.
* **Memory Limit:** Exactly 2 GiB maximum per run.
* **Concurrency:** At most 1 run can be executed at any given time (absolutely no parallelization or concurrent testing threads).
* **Test Data Location:** You can find benchmark files to test and evaluate your translator implementation against under @misc/tests/benchmarks/.

## Ready to Begin
Acknowledge that you understand the task. Before writing any implementation code, we must go through a mandatory brainstorming to remove any ambiguity.
```

Lots of things in the process were *ad hoc*, including the prompt. A few are worth explaining: I was running Claude Code in the background and I did not want it disturbing my work, so I limited it to 2 GiB per run and 120 seconds per task, without parallelism.

In the brainstorming phase, Claude asked if it should convert file-by-file and then stop for testing, and what the workflow should look like. I told it to implement the entire translator in one shot and to use the following workflow:

1. Claude implemented something and stopped whenever it believed to be done
2. I evaluated the implementation on held-out benchmarks (i.e., a subset of IPC domains from the [downward-benchmarks repo](https://github.com/aibasel/downward-benchmarks/), and the [hard-to-ground domains benchmark](https://github.com/abcorrea/htg-domains/)) on a cluster (using 8 GiB and 5 minutes per task)
3. I gave the [lab](https://lab.readthedocs.io/en/stable/) report back to Claude
4. Claude tried to fix any unexpected behavior or performance problems

(I used a single agent spawned from the terminal --- no sub-agents, no orchestration, no `/goal`, no Ralph loop. The only global skill I have installed is the [`uv` skill by Armin Ronacher](https://github.com/mitsuhiko/agent-stuff/tree/main/skills/uv) but this was only loaded later, when it created the lab scripts. Claude was also told to not change the algorithms conceptually.)

During early development, Claude only had access to the small benchmark that is [shipped with Fast Downward](https://github.com/aibasel/downward/tree/main/misc/tests/benchmarks). Later, I augmented the benchmark suite from Fast Downward's repo to include more challenging tasks, because Claude did not have enough tasks to estimate the performance of its own implementation.

When it had problems, I tried to not tell Claude what it had done wrong or what should be fixed. Evidently, this is an extreme approach and it would not be what anyone would do in a real scenario. There were a few exceptions where I did have to intervene: a few times Claude was too defeatist ("*oh there is nothing we can do, we will never outperform the Python implementation*") and I had to ask it things like "*What if you stop doing string comparisons and use indices instead?*"; on the other hand, Claude often decided to simply not implement certain ideas. For example, in the invariant synthesis algorithm, Claude ignored the variable ordering implemented in the translator (using the weighted causal graph) and went with something on its own --- and it did not work properly. It took a few iterations until it finally implemented the same algorithm.

The results are available online (for [the IPC](https://abcorrea.github.io/assets/tmp/claude-translator-ipc.html) and for [the hard-to-ground](https://abcorrea.github.io/assets/tmp/claude-translator-htg.html) domains). There are two columns per implementation: `-h2-lama-first` runs the full pipeline with invariant synthesis on; `-no-h2-lama-first` turns it off (`--invariant-generation-max-candidates 0`). When the invariant synthesis is off, the implementations produce the same task; when it is on, Claude failed to make the C++ implementation identical. The Mersenne Twister inside the invariant synthesis is deterministic on both sides: same seed (314159), same algorithm name. But *apparently* libstdc++'s `mt19937` plus `uniform_int_distribution` produces a different sequence than CPython's `random.Random.randrange`. Claude never managed to find a workaround to make both identical.

The speed improvement is quite substantial but, overall, I would expect that a good C++ port would give us a more robust improvement in number of solved tasks. The gain in coverage for the hard-to-ground benchmark was 12%, despite the substantial speed up in translation time. For the IPC domains, the coverage change is negligible. For context, our [2023 paper](https://abcorrea.github.io/assets/pdf/correa-et-al-icaps2023.pdf) led to 7% gain in coverage, despite only reimplementing the grounder component.

A few things never got fixed. The mismatch in the variable ordering in the invariant synthesis is the obvious one. And the significant memory improvements only happened after I explicitly suggested swapping string arguments for indices. Based on the conversation, Claude wouldn't have proposed that refactor on its own.

What about code quality? I would be lying if I said I read all the code. But the parts I've seen (mostly the [grounder component](https://github.com/abcorrea/downward-claude-translator/tree/main/src/translate-cpp/grounding)) were more readable than I expected, although I would have done things differently in terms of data structures and data flow. The code seems to me a bit Pythonic sometimes, but this is perhaps an issue with my instructions.

And how long did it take? Tough to say because this was not my priority, so often I left Claude hanging for a few hours. I also have Claude Code Pro, so the workload is "naturally" split into 5-hours chunks. If we assume that Claude was working on it from it the first commit until the last one in each session, then the entire project took around 26--27 hours (I am accounting some "buffer time" for the work done before the first commit, and also including the time taken for the benchmark evaluation).
