## 1. Experimental Results

Running the DP algorithm on `egfr_myc_large_network.csv` produced:

| Metric | Result |
|---|---|
| Minimum signal latency from EGFR to MYC | 7.5 minutes |
| Number of co-optimal minimal-latency pathways | 4 |
| Intermediate gene bottlenecks | AKT1, PDK1, PIK3CA |

All four minimal-latency pathways pass through AKT1, PDK1, and PIK3CA. For each of these genes, the number of shortest paths passing through it, `C[v] * R[v]`, equals the total shortest-path count of 4.

These genes are bottlenecks among the shortest pathways; slower alternative pathways may bypass them. The dataset is synthetic, so these results should not be interpreted as evidence for real therapeutic targets.

## 2. Near-Optimal Signaling Dynamics

Let `L* = 7.5` minutes be the optimal latency and `delta` be the allowed additional latency. We want to count all pathways with total latency at most `L* + delta`.

Instead of storing only the minimum distance and its path count, I would store a latency-indexed count at each node:

    H[v][t] = number of paths from EGFR to v with total latency t

Initialize `H[EGFR][0] = 1`, with all other counts initially zero. Process nodes in topological order. For every edge `u -> v` with weight `w`, propagate each stored arrival time:

    H[v][t + w] += H[u][t]

Because edge weights are positive, states exceeding `L* + delta` can be discarded. The final answer is:

    sum(H[MYC][t] for t <= L* + delta)

For example, if `delta = 0.2` minutes, the algorithm counts all pathways taking at most 7.7 minutes.

Simple path-counting is insufficient because paths reaching the same node can have different arrival times and therefore different remaining latency budgets. Keeping only the shortest arrival discards potentially valid near-optimal paths, while combining all arrivals into one count loses the information needed to enforce the threshold.

The additional state must therefore track arrival time, or equivalently the remaining latency budget, together with its path count. Fixed-precision weights can be scaled into integer time units for exact indexing. For arbitrary real-valued weights, storing all distinct arrival times can require exponentially many states; time bins offer an approximation but can affect classification near the threshold.

## 3. Graph DP vs. Grid DP

Topological ordering and nested matrix loops both ensure that dependencies are computed before the states that depend on them.

In Smith-Waterman, each cell depends on its upper, left, and upper-left neighbors. Row-by-row nested loops ensure that these cells have already been evaluated.

A gene network has an irregular dependency structure rather than a rectangular grid. Topological ordering provides the corresponding evaluation schedule: every upstream parent is processed before its downstream child.

Thus, both approaches allow each DP state to use previously computed results without repeatedly solving the same subproblems.

## 4. Biological Feedback Loops

A directed cycle prevents a complete topological ordering. Nodes within the cycle retain incoming dependencies from one another, so Kahn's algorithm eventually empties its queue before processing all nodes. The provided implementation detects this and raises a cycle error.

However, with positive edge weights, traversing a cycle always increases latency. Removing that cycle produces a shorter route, so a shortest route never needs to repeat a cycle. If MYC is reachable from EGFR, a finite shortest distance therefore remains well-defined.

DAG DP fails because a cycle prevents the required dependency ordering. An algorithm such as Dijkstra's can still compute shortest distances in a graph with positive edge weights, even when cycles are present.
