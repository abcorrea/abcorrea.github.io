---
layout: page
permalink: /leaderboard/
title: leaderboard
description: Coverage of LLM-generated heuristics on the IPC 2023 Learning Track domains.
nav: true
pretty_table: true
---

This leaderboard shows the performance of LLM-generated heuristics for traditional PDDL planning domains. Our [original
paper](../assets/pdf/correa-et-al-neurips2025.pdf) only contained models released until mid-2025. We created this leaderboard to evaluate the performance of frontier models as they are released. 

Coverage (number of solved tasks) of LLM-generated
heuristic functions evaluated using greedy best-first search in
[Pyperplan](https://doi.org/10.5281/zenodo.3700819). Each heuristic is
selected from a pool of 25 candidates based on training set performance. 

All domains have 90 test
tasks (720 total), following the easy/medium/hard split used in the IPC 2023. Each run is limited to 30 minutes and 8 GiB. Test tasks are
out-of-distribution. See [our NeurIPS 2025
paper](../assets/pdf/correa-et-al-neurips2025.pdf) for more details.

Best values per domain are shown in <strong>bold</strong>. Entries in <em>italics</em> are
baseline planners (not LLM-generated heuristics).

<blockquote class="block-warning">
<h5>Last Update (2026-02-21)</h5>
  <p style="font-size: 1rem;">Added results for GPT 5.2, Gemini 3 Pro (Preview) and Gemini 3.1 Pro (Preview).</p>
</blockquote>

<br>

<script>
function numericSort(a, b) {
  var aa = parseInt(a.replace(/<[^>]*>/g, ''));
  var bb = parseInt(b.replace(/<[^>]*>/g, ''));
  return aa - bb;
}
</script>

<style>
#leaderboard-table {
  font-size: 0.5rem;
}
#leaderboard-table th:not(:first-child),
#leaderboard-table td:not(:first-child) {
  text-align: right;
}
#leaderboard-table td:first-child,
#leaderboard-table th:first-child {
  white-space: nowrap;
}
</style>

<table class="table table-hover table-sm" id="leaderboard-table" data-toggle="table" data-sortable="true">
  <thead>
    <tr>
      <th data-sortable="true">Model</th>
      <th data-sortable="true" data-sorter="numericSort">Total</th>
      <th data-sortable="true" data-sorter="numericSort">Block.</th>
      <th data-sortable="true" data-sorter="numericSort">Child.</th>
      <th data-sortable="true" data-sorter="numericSort">Floor.</th>
      <th data-sortable="true" data-sorter="numericSort">Mico.</th>
      <th data-sortable="true" data-sorter="numericSort">Rov.</th>
      <th data-sortable="true" data-sorter="numericSort">Soko.</th>
      <th data-sortable="true" data-sorter="numericSort">Span.</th>
      <th data-sortable="true" data-sorter="numericSort">Trans.</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Gemini 3.1 Pro (Preview)</td>
      <td><strong>460</strong></td>
      <td><strong>84</strong></td>
      <td>58</td>
      <td><strong>30</strong></td>
      <td>83</td>
      <td>53</td>
      <td>31</td>
      <td>66</td>
      <td>55</td>
    </tr>
    <tr>
      <td>GPT-5</td>
      <td>450</td>
      <td>79</td>
      <td><strong>61</strong></td>
      <td>12</td>
      <td><strong>90</strong></td>
      <td>40</td>
      <td>30</td>
      <td>68</td>
      <td><strong>70</strong></td>
    </tr>
    <tr>
      <td><em>Fast Down. Stone Soup'23</em></td>
      <td>448</td>
      <td>58</td>
      <td>45</td>
      <td>23</td>
      <td><strong>90</strong></td>
      <td>64</td>
      <td><strong>40</strong></td>
      <td>64</td>
      <td>64</td>
    </tr>
    <tr>
      <td>Gemini 3 Pro (Preview)</td>
      <td>426</td>
      <td><strong>84</strong></td>
      <td>60</td>
      <td>12</td>
      <td>88</td>
      <td>33</td>
      <td>27</td>
      <td>66</td>
      <td>56</td>
    </tr>
    <tr>
      <td>GPT-5.2</td>
      <td>407</td>
      <td>73</td>
      <td>58</td>
      <td>10</td>
      <td>75</td>
      <td>35</td>
      <td>31</td>
      <td>66</td>
      <td>59</td>
    </tr>
    <tr>
      <td><em>LAMA</em></td>
      <td>396</td>
      <td>55</td>
      <td>35</td>
      <td>12</td>
      <td><strong>90</strong></td>
      <td><strong>68</strong></td>
      <td><strong>40</strong></td>
      <td>30</td>
      <td>66</td>
    </tr>
    <tr>
      <td>o1</td>
      <td>373</td>
      <td>29</td>
      <td>60</td>
      <td>9</td>
      <td><strong>90</strong></td>
      <td>40</td>
      <td>32</td>
      <td>69</td>
      <td>44</td>
    </tr>
    <tr>
      <td>DeepSeek R1</td>
      <td>373</td>
      <td>66</td>
      <td>22</td>
      <td>4</td>
      <td><strong>90</strong></td>
      <td>32</td>
      <td>30</td>
      <td><strong>70</strong></td>
      <td>59</td>
    </tr>
    <tr>
      <td>Gemini 2.5 Pro</td>
      <td>361</td>
      <td>52</td>
      <td><strong>61</strong></td>
      <td>4</td>
      <td>89</td>
      <td>38</td>
      <td>30</td>
      <td>30</td>
      <td>57</td>
    </tr>
    <tr>
      <td>o3</td>
      <td>354</td>
      <td>36</td>
      <td>45</td>
      <td>11</td>
      <td>76</td>
      <td>36</td>
      <td>27</td>
      <td>66</td>
      <td>57</td>
    </tr>
    <tr>
      <td>DeepSeek V3</td>
      <td>343</td>
      <td>45</td>
      <td>55</td>
      <td>3</td>
      <td>64</td>
      <td>34</td>
      <td>31</td>
      <td>69</td>
      <td>42</td>
    </tr>
    <tr>
      <td>GPT-4.1</td>
      <td>327</td>
      <td>54</td>
      <td>59</td>
      <td>11</td>
      <td>38</td>
      <td>30</td>
      <td>27</td>
      <td>63</td>
      <td>45</td>
    </tr>
    <tr>
      <td>Gemini 2.0 Flash Think.</td>
      <td>305</td>
      <td>37</td>
      <td>14</td>
      <td>8</td>
      <td>88</td>
      <td>39</td>
      <td>32</td>
      <td>30</td>
      <td>57</td>
    </tr>
    <tr>
      <td>Gemini 2.0 Flash</td>
      <td>296</td>
      <td>35</td>
      <td>32</td>
      <td>4</td>
      <td><strong>90</strong></td>
      <td>32</td>
      <td>31</td>
      <td>30</td>
      <td>42</td>
    </tr>
    <tr>
      <td>GPT-4o</td>
      <td>258</td>
      <td>35</td>
      <td>24</td>
      <td>3</td>
      <td>63</td>
      <td>32</td>
      <td>28</td>
      <td>30</td>
      <td>43</td>
    </tr>
    <tr>
      <td>DeepSeek R1 DistQwen14B</td>
      <td>214</td>
      <td>34</td>
      <td>16</td>
      <td>3</td>
      <td>30</td>
      <td>32</td>
      <td>24</td>
      <td>30</td>
      <td>45</td>
    </tr>
    <tr>
      <td>Qwen3 4B Instruct</td>
      <td>208</td>
      <td>28</td>
      <td>13</td>
      <td>2</td>
      <td>48</td>
      <td>19</td>
      <td>25</td>
      <td>30</td>
      <td>43</td>
    </tr>
    <tr>
      <td>Gemma 3 12B</td>
      <td>187</td>
      <td>30</td>
      <td>12</td>
      <td>1</td>
      <td>30</td>
      <td>19</td>
      <td>31</td>
      <td>30</td>
      <td>34</td>
    </tr>
  </tbody>
</table>

<br>


<blockquote class="block-danger">
  <p style="font-size: 0.9rem;">If you would like us to test another model, please contact us directly via email.</p>
</blockquote>
