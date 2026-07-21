# vani-discrete — TODO

> Compiler builtins that already exist and must NOT be reimplemented:
> `abs` `i64_test_bit` `i64_factorial` `i64_binomial` `f64_inf()`
> `push` `pop` `len` `set` `vec`
>
> Also already covered by the compiler's builtin `Graph` type (opaque from
> vāṇी source, see README for why this package can't build on top of it):
> `graph_new` `graph_add_edge` `graph_bfs_reach` `graph_dfs_reach`
> `graph_dijkstra` `graph_has_cycle` `graph_mst_kruskal` `graph_mst_prim`
> `graph_astar` `graph_topo_sort`
>
> No Kosh dependencies -- v0.1.0 is a standalone package.

---

## v0.1.0 — Implemented ✓

Fills the G1-G7 items from kosh-index/ROADMAP.md's "Known gaps within
'mostly done' rows" (Discrete math).

### Adjacency matrix construction (3 functions)
- [x] `disc_adj_new`, `disc_adj_add_edge`, `disc_adj_add_edge_undirected`

### G1: All-pairs shortest path (1 function)
- [x] `disc_floyd_warshall` -- validated against a hand-computed 4-node
      weighted graph (including an unreachable-pair case returning
      `f64_inf()`)

### G2: Strongly-connected components (1 function)
- [x] `disc_scc_kosaraju` -- two-pass DFS + transpose, both DFS passes
      **iterative** (explicit Vec-based stack, not language recursion).
      Validated against a 5-node graph with two known SCCs

### G3: Max-flow / min-cut (2 functions)
- [x] `disc_max_flow`, `disc_min_cut_nodes` -- Edmonds-Karp (BFS
      shortest-augmenting-path). Validated against the classic CLRS
      max-flow example (flow = 23), plus a composed check: the min cut's
      capacity, summed from the *original* capacity matrix, equals the max
      flow -- the max-flow-min-cut theorem, verified rather than assumed

### G4: Bipartite matching (1 function)
- [x] `disc_bipartite_matching` -- Kuhn's augmenting-path algorithm
      (recursive DFS per left-node, `#[bounded(256)]` since recursion
      depth is bounded by `n_right`). Validated against a 2x2 case that
      *requires* an augmenting-path swap to reach the true maximum (naive
      greedy without augmenting would under-match)

### G5: Graph coloring (1 function)
- [x] `disc_greedy_coloring` -- greedy by node index, NOT exact/minimum
      (exact graph coloring is NP-hard). Validated against a 4-cycle
      (2-colorable) and a triangle (needs 3 colors)

### G6: Combinatorics enumeration (3 functions)
- [x] `disc_next_permutation` -- Narayana's algorithm (same as C++
      `std::next_permutation`). Composed check: total permutations
      generated for n=4 equals `i64_factorial(4)`
- [x] `disc_next_combination` -- lexicographic k-combination stepping.
      Composed check: total combinations for (6,3) equals `i64_binomial(6,3)`
- [x] `disc_subset_from_bitmask` -- decode a bitmask via `i64_test_bit`.
      Composed check: summing subset sizes over every mask 0..2^4-1 equals
      `n * 2^(n-1)` (each element appears in exactly half of all subsets)

### G7: Integer partitions (1 function)
- [x] `disc_partition_count` -- O(n²) DP for p(n), validated against known
      values (p(6)=11, p(7)=15, ...)

### Tests and examples
- [x] `tests/test_graph_paths.vani` -- Floyd-Warshall, Kosaraju SCC
- [x] `tests/test_flow_matching.vani` -- max-flow, the min-cut composed
      theorem check, bipartite matching requiring real augmentation
- [x] `tests/test_coloring_combinatorics.vani` -- coloring, both
      enumeration composed checks against builtins, subset-sum identity,
      partition counts
- [x] `examples/network_analysis_demo.vani` -- shortest paths + SCC +
      max-flow/min-cut on one small network
- [x] `examples/assignment_and_routing_demo.vani` -- worker-task bipartite
      assignment, brute-force shortest round trip via permutation
      enumeration

### Safety annotations
- [x] `#[bounded_stack(bytes=N)]` on every function, budgets set to `vanic
      check`'s exact reported worst-case (largest: `disc_bipartite_matching`
      at 25876 bytes, dominated by `_disc_bip_try_augment`'s
      `#[bounded(256)]` recursion cap -- 256 * ~100 bytes/frame)
- [x] `#[bounded(256)]` on `_disc_bip_try_augment`, the only recursive
      function in this library (recursion depth bounded by `n_right`,
      capped to match "modest size" scope); Kosaraju's two DFS passes are
      iterative specifically to avoid needing a similar bound

---

## Future

No v0.2.0 is currently planned. Candidates if a concrete need shows up:
Tarjan's SCC (single-pass, avoids the transpose -- only worth it if
Kosaraju's two-pass cost becomes a real bottleneck), Dinic's or
push-relabel max-flow (better asymptotic complexity than Edmonds-Karp),
Hopcroft-Karp bipartite matching, an exact/optimal graph coloring for
small graphs (backtracking with pruning), and integer partition
*enumeration* (not just counting) if a concrete use case shows up --
deliberately deferred, see README's scope-decisions section.
