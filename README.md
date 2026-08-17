# vani-discrete

Graph algorithms and combinatorics enumeration library for the
[vāṇी compiler](https://github.com/enthusiasticgeek/vani-compiler).

No Kosh dependencies -- v0.1.0 is a standalone package.

**API reference / tutorial:** <https://enthusiasticgeek.github.io/vani-discrete/>

## Add to your project

```toml
# vani.toml
[deps]
discrete = { registry = "kosh", version = "^0.1" }
```

```sh
vanic add discrete
vanic build
```

## Why not the compiler's builtin Graph type?

The compiler ships builtin graph functions (`graph_new`/`add_edge`/
`bfs_reach`/`dijkstra`/`mst_kruskal`/`mst_prim`/`astar`/`topo_sort`, call
these directly -- do not duplicate them here). But the builtin `Graph` type
is **opaque** from vāṇी source: there's no accessor to enumerate a node's
neighbors or list all edges, so algorithms beyond that fixed set (all-pairs
shortest path, strongly-connected components, max-flow, bipartite matching,
graph coloring) can't be built on top of it. This package uses its own flat
row-major `Vec<f64>` adjacency-matrix encoding instead (`n*n`, entry
`i*n+j` = edge weight from `i` to `j`, `f64_inf()` = no edge) -- the same
"no hidden metadata" convention every other kosh package uses.

## What's included (v0.1.0 — complete; see TODO.md)

| Module | Functions |
|---|---|
| Adjacency matrix construction | `disc_adj_new`, `disc_adj_add_edge`, `disc_adj_add_edge_undirected` |
| All-pairs shortest path (Floyd-Warshall) | `disc_floyd_warshall` |
| Strongly-connected components (Kosaraju) | `disc_scc_kosaraju` |
| Max-flow / min-cut (Edmonds-Karp) | `disc_max_flow`, `disc_min_cut_nodes` |
| Bipartite matching (Kuhn's algorithm) | `disc_bipartite_matching` |
| Graph coloring (greedy) | `disc_greedy_coloring` |
| Combinatorics enumeration | `disc_next_permutation`, `disc_next_combination`, `disc_subset_from_bitmask` |
| Integer partitions | `disc_partition_count` |

## Scope decisions (read before using)

- **Simplest correct algorithm, not the asymptotically optimal one** --
  matching this ecosystem's precedent (e.g. vani-geometry's O(n²) convex
  hull over divide-and-conquer). Floyd-Warshall not Johnson's, Kosaraju not
  Tarjan, Edmonds-Karp not Dinic's, Kuhn's not Hopcroft-Karp.
- **Graph coloring is greedy, not exact** -- gives *a* valid coloring, not
  necessarily the minimum number of colors (exact graph coloring is
  NP-hard).
- **Combinatorics is enumeration, not materialization** -- `disc_next_permutation`
  / `disc_next_combination` advance one step in place and the caller loops
  on the boolean return, matching the C++ `std::next_permutation`
  convention. Deliberately not a function returning every permutation in a
  nested `Vec<Vec<i64>>` -- that would blow up memory for large `n`, and
  this ecosystem avoids nested-Vec encodings throughout (vani-tensor's flat
  encoding is the precedent).
- **Integer partitions are counted, not enumerated** -- `disc_partition_count`
  computes p(n), the literal "partition function" from number theory.
  Partition *enumeration* algorithms are genuinely fiddly to get exactly
  right without a symbolic reference to check against (the same
  risk-avoidance call as skipping a hand-derived Ferrari's-method quartic
  in vani-algebra).

## Correctness

Beyond hand-computed/textbook examples (the CLRS max-flow network, a
4-cycle vs. a triangle for coloring), two composed checks tie functions
together: the **max-flow-min-cut theorem** is verified, not assumed (the
min cut's capacity, summed from the *original* capacity matrix using
`disc_min_cut_nodes`'s output, must equal `disc_max_flow`'s flow value),
and permutation/combination enumeration totals are cross-checked against
the compiler's own `i64_factorial`/`i64_binomial` builtins. See `tests/`.

## What this library does NOT provide

These are already vāṇी compiler builtins — call them directly, no import needed:

`abs` `i64_test_bit` `i64_factorial` `i64_binomial` `f64_inf()`
`push` `pop` `len` `set` `vec`

Also already covered by the compiler's builtin `Graph` type (see above):
`graph_new` `graph_add_edge` `graph_bfs_reach` `graph_dfs_reach`
`graph_dijkstra` `graph_has_cycle` `graph_mst_kruskal` `graph_mst_prim`
`graph_astar` `graph_topo_sort`

## License

MIT
