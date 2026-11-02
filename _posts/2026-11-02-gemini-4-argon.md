---
layout: post
title: Evaluating Gemini 4 Argon on End-to-End Planning
date: 2026-11-02 12:00:00 +0000
description: Comparing Gemini 4 Argon as an end-to-end planner with Gemini 3.1 Pro and with LLM-generated heuristics.
tags: [planning, llms, experiments]
categories: [llm-planning]
chart:
  echarts: true
---

<style>
  .chart-short + .echarts { height: 260px; }
  .chart-tall + .echarts { height: 520px; }
</style>

Google made [Gemini 4
Argon](https://deepmind.google/models/model-cards/gemini-4-argon/) available in
the Gemini API in October. I tested it as an end-to-end planner: the model
receives a PDDL domain and a PDDL task, and it has to return a plan. This post
only reports coverage, i.e., the percentage of solved tasks. I do not discuss
time, cost, or plan quality. I compare Argon with Gemini 3.1 Pro in the same
setting, and with the LLM-generated heuristics from our [NeurIPS 2026
paper](https://abcorrea.github.io/assets/pdf/pereira-et-al-arxiv2026.pdf).

I used the same setup as in our paper [Frontier Large Language Models Rival
State-of-the-Art Planners](https://arxiv.org/abs/2511.09378). The prompt
contains the domain, the task, and two examples (a Gripper task and a Logistics
task, each with its plan). The model has a single attempt and no tools, so it
cannot run code or call a planner. Every plan is validated with
[VAL](https://github.com/KCL-Planning/VAL), and a task counts as solved only if
VAL accepts the plan. For Argon, I set the thinking level to high, which is the
default for Gemini 3.1 Pro (Argon's default is medium). Argon ignores
temperature and top-P, so I did not set them. Both models used the default
output limit of 65,536 tokens.

I used three sets of tasks:

1. The *standard* set contains eight domains of the [IPC 2023 learning track](https://github.com/ipc2023-learning/benchmarks). We generated 45 new tasks per domain with the competition generators (15 easy, 15 medium, and 15 hard), so the set has 360 tasks.
2. The *obfuscated* set has the same 360 tasks, but all names in the domain and task files are replaced by meaningless symbols, such as `p1` for a predicate, `a1` for an action, and `o1` for an object. Comments are removed.
3. The third set has two new domains that were created during the NeurIPS 2026 rebuttal phase. In *cisterns*, tanks connected by hoses exchange water one unit at a time, but water only flows from a tank with at least two units more than the destination. In *orrery*, twisting the coupling between two wheels moves one wheel forward by one mark and the other backward by one mark, and the goal is to bring every wheel back to its reference mark.

The standard and obfuscated sets are the ones from our paper on frontier LLMs. The two new domains were created in June 2026, after Argon's knowledge cutoff (March 2026, according to its model card).

The figure below shows the coverage per domain for the two IPC sets.

```echarts
{
  "backgroundColor": "transparent",
  "legend": {"top": 0, "left": 0},
  "tooltip": {"trigger": "axis", "axisPointer": {"type": "shadow"}},
  "title": [
    {"text": "Standard", "left": "22%", "top": 32, "textStyle": {"fontSize": 13, "fontWeight": "normal"}},
    {"text": "Obfuscated", "left": "63%", "top": 32, "textStyle": {"fontSize": 13, "fontWeight": "normal"}}
  ],
  "grid": [
    {"left": "22%", "width": "32%", "top": 60, "bottom": 30},
    {"left": "63%", "width": "32%", "top": 60, "bottom": 30}
  ],
  "xAxis": [
    {"gridIndex": 0, "type": "value", "min": 0, "max": 100, "interval": 50, "axisLabel": {"formatter": "{value}%"}, "splitLine": {"lineStyle": {"type": "solid", "color": "rgba(128, 128, 128, 0.25)"}}},
    {"gridIndex": 1, "type": "value", "min": 0, "max": 100, "interval": 50, "axisLabel": {"formatter": "{value}%"}, "splitLine": {"lineStyle": {"type": "solid", "color": "rgba(128, 128, 128, 0.25)"}}}
  ],
  "yAxis": [
    {"gridIndex": 0, "type": "category", "inverse": true, "axisTick": {"show": false},
     "data": ["blocksworld", "childsnack", "floortile", "miconic", "rovers", "sokoban", "spanner", "transport", "All domains"]},
    {"gridIndex": 1, "type": "category", "inverse": true, "axisTick": {"show": false}, "axisLabel": {"show": false},
     "data": ["blocksworld", "childsnack", "floortile", "miconic", "rovers", "sokoban", "spanner", "transport", "All domains"]}
  ],
  "series": [
    {"name": "Gemini 4 Argon", "type": "bar", "xAxisIndex": 0, "yAxisIndex": 0, "barGap": "15%", "barCategoryGap": "35%",
     "itemStyle": {"color": "#3987e5", "borderRadius": [0, 3, 3, 0]},
     "data": [100, 100, 93, 98, 62, 56, 100, 98, 88]},
    {"name": "Gemini 3.1 Pro", "type": "bar", "xAxisIndex": 0, "yAxisIndex": 0, "barGap": "15%", "barCategoryGap": "35%",
     "itemStyle": {"color": "#d95926", "borderRadius": [0, 3, 3, 0]},
     "data": [76, 98, 56, 80, 36, 33, 100, 67, 68]},
    {"name": "Gemini 4 Argon", "type": "bar", "xAxisIndex": 1, "yAxisIndex": 1, "barGap": "15%", "barCategoryGap": "35%",
     "itemStyle": {"color": "#3987e5", "borderRadius": [0, 3, 3, 0]},
     "data": [100, 100, 84, 98, 58, 44, 100, 98, 85]},
    {"name": "Gemini 3.1 Pro", "type": "bar", "xAxisIndex": 1, "yAxisIndex": 1, "barGap": "15%", "barCategoryGap": "35%",
     "itemStyle": {"color": "#d95926", "borderRadius": [0, 3, 3, 0]},
     "data": [69, 89, 40, 78, 33, 31, 98, 76, 64]}
  ]
}
```

On the standard set, Argon solves 88% of the tasks and Gemini 3.1 Pro solves
68%. Argon solves almost every task that 3.1 Pro solves. Just two tasks (0.6%)
are solved by 3.1 Pro and not by Argon (one in miconic, one in transport). In
contrast, 21% of the tasks are solved by Argon and not by 3.1 Pro. The gap grows
with the size of the tasks. Both models solve nearly all easy tasks (100% and
99%), while Argon solves 91% of the medium tasks and 74% of the hard tasks,
against 72% and 33% for 3.1 Pro.

(Nine of the 15 hard rovers tasks have prompts longer than the 1M-token context
window, so neither model can attempt them, and I count them as failures. Neither
model solves any hard sokoban task.)

On the obfuscated set, Argon solves 85% of the tasks, three percentage points
lower than on the standard set. Gemini 3.1 Pro solves 64%. Moreover, Gemini 3.1 Pro
solves no task that Argon fails to solve, and 21% of the tasks are solved by
Argon and not by 3.1 Pro. *Argon's coverage on the obfuscated set is also higher
than 3.1 Pro's coverage on the standard set* (85% vs. 68%).

The figure below shows the coverage by difficulty level on both sets.

```echarts
{
  "backgroundColor": "transparent",
  "legend": {"top": 0, "left": 0},
  "tooltip": {"trigger": "axis", "axisPointer": {"type": "shadow"}},
  "title": [
    {"text": "Standard", "left": "22%", "top": 32, "textStyle": {"fontSize": 13, "fontWeight": "normal"}},
    {"text": "Obfuscated", "left": "63%", "top": 32, "textStyle": {"fontSize": 13, "fontWeight": "normal"}}
  ],
  "grid": [
    {"left": "22%", "width": "32%", "top": 60, "bottom": 30},
    {"left": "63%", "width": "32%", "top": 60, "bottom": 30}
  ],
  "xAxis": [
    {"gridIndex": 0, "type": "value", "min": 0, "max": 100, "interval": 50, "axisLabel": {"formatter": "{value}%"}, "splitLine": {"lineStyle": {"type": "solid", "color": "rgba(128, 128, 128, 0.25)"}}},
    {"gridIndex": 1, "type": "value", "min": 0, "max": 100, "interval": 50, "axisLabel": {"formatter": "{value}%"}, "splitLine": {"lineStyle": {"type": "solid", "color": "rgba(128, 128, 128, 0.25)"}}}
  ],
  "yAxis": [
    {"gridIndex": 0, "type": "category", "inverse": true, "axisTick": {"show": false}, "data": ["Easy", "Medium", "Hard"]},
    {"gridIndex": 1, "type": "category", "inverse": true, "axisTick": {"show": false}, "axisLabel": {"show": false}, "data": ["Easy", "Medium", "Hard"]}
  ],
  "series": [
    {"name": "Gemini 4 Argon", "type": "bar", "xAxisIndex": 0, "yAxisIndex": 0, "barGap": "15%", "barMaxWidth": 16,
     "itemStyle": {"color": "#3987e5", "borderRadius": [0, 3, 3, 0]},
     "data": [100, 91, 74]},
    {"name": "Gemini 3.1 Pro", "type": "bar", "xAxisIndex": 0, "yAxisIndex": 0, "barGap": "15%", "barMaxWidth": 16,
     "itemStyle": {"color": "#d95926", "borderRadius": [0, 3, 3, 0]},
     "data": [99, 72, 33]},
    {"name": "Gemini 4 Argon", "type": "bar", "xAxisIndex": 1, "yAxisIndex": 1, "barGap": "15%", "barMaxWidth": 16,
     "itemStyle": {"color": "#3987e5", "borderRadius": [0, 3, 3, 0]},
     "data": [100, 86, 70]},
    {"name": "Gemini 3.1 Pro", "type": "bar", "xAxisIndex": 1, "yAxisIndex": 1, "barGap": "15%", "barMaxWidth": 16,
     "itemStyle": {"color": "#d95926", "borderRadius": [0, 3, 3, 0]},
     "data": [98, 64, 30]}
  ]
}
```
{: .chart-short}

How does this compare with LLM-generated heuristics? In our [NeurIPS 2026
paper](https://abcorrea.github.io/assets/pdf/pereira-et-al-arxiv2026.pdf), the
LLM writes one heuristic function per domain using small training tasks and
refines it with a CEGIS loop, and
[Pyperplan](https://doi.org/10.5281/zenodo.3700819) then uses this heuristic to
solve the test tasks.

The CEGIS loop is used to enforce that the heuristic satisfies a given
*property* in the small training tasks. In the NeurIPS 2026 paper, we enforce
the heuristics to be *direct*, i.e., leading directly to a goal through
hill-climbing (HC). Its heuristics were written by Gemini 3.1 Pro, and the
planner uses them with HC. *FF* is the classical FF heuristic with greedy
best-first search (GBFS).

```echarts
{
  "backgroundColor": "transparent",
  "legend": {"top": 0, "left": 0},
  "tooltip": {"trigger": "axis", "axisPointer": {"type": "shadow"}},
  "grid": {"left": "22%", "right": "5%", "top": 60, "bottom": 30},
  "xAxis": {"type": "value", "min": 0, "max": 100, "interval": 25, "axisLabel": {"formatter": "{value}%"}, "splitLine": {"lineStyle": {"type": "solid", "color": "rgba(128, 128, 128, 0.25)"}}},
  "yAxis": {"type": "category", "inverse": true, "axisTick": {"show": false},
            "data": ["blocksworld", "childsnack", "floortile", "miconic", "rovers", "sokoban", "spanner", "transport", "All domains"]},
  "series": [
    {"name": "Argon, end-to-end", "type": "bar", "barGap": "15%", "barCategoryGap": "30%",
     "itemStyle": {"color": "#3987e5", "borderRadius": [0, 3, 3, 0]},
     "data": [100, 100, 93, 98, 62, 56, 100, 98, 88]},
    {"name": "3.1 Pro, end-to-end", "type": "bar", "barGap": "15%", "barCategoryGap": "30%",
     "itemStyle": {"color": "#d95926", "borderRadius": [0, 3, 3, 0]},
     "data": [76, 98, 56, 80, 36, 33, 100, 67, 68]},
    {"name": "Direct (HC)", "type": "bar", "barGap": "15%", "barCategoryGap": "30%",
     "itemStyle": {"color": "#199e70", "borderRadius": [0, 3, 3, 0]},
     "data": [86, 80, 45, 100, 47, 33, 83, 76, 69]},
    {"name": "FF (GBFS)", "type": "bar", "barGap": "15%", "barCategoryGap": "30%",
     "itemStyle": {"color": "#c98500", "borderRadius": [0, 3, 3, 0]},
     "data": [26, 13, 12, 68, 32, 32, 33, 26, 30]}
  ]
}
```
{: .chart-tall}

Over these eight domains, Direct solves 69% of the tasks, about the same as 3.1
Pro end-to-end (68%). Argon end-to-end solves 88%. It has the highest coverage
in every domain except miconic, where Direct solves all tasks and Argon solves
98% (in spanner, it ties with 3.1 Pro end-to-end). In sokoban, Direct and 3.1
Pro end-to-end both solve 33% of the tasks, and Argon solves 56%. In rovers, the
nine prompts that do not fit in the context window limit end-to-end coverage to
at most 80%, while the heuristics do not have this limit.

For cisterns and orrery (the new domains!), these are the results:

| Domain   | Direct (HC) | FF (GBFS) | Argon, end-to-end |
|----------|------------:|----------:|------------------:|
| cisterns |         30% |       34% |           **91%** |
| orrery   |    **100%** |       64% |          **100%** |

On orrery, Direct and Argon solve every task. On cisterns, Direct solves 30% of
the tasks, fewer than FF (34%), and Argon solves 91%.

Two years ago, the motivation to *write code* instead of doing end-to-end
planning was that (1) it was *cheaper* and (2) it had *better performance*. It
is not obvious that both are still true --- although code synthesis is
definitely still cheaper than end-to-end.

If open models continue to improve, who knows...
