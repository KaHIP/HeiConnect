HeiConnect v1.0
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![C++](https://img.shields.io/badge/C++-17-blue.svg)](https://isocpp.org/)
[![CMake](https://img.shields.io/badge/CMake-3.14+-064F8C.svg)](https://cmake.org/)
[![Linux](https://img.shields.io/badge/Linux-supported-success.svg)](https://github.com/KaHIP/HeiConnect)
[![GitHub Stars](https://img.shields.io/github/stars/KaHIP/HeiConnect)](https://github.com/KaHIP/HeiConnect/stargazers)
[![GitHub Issues](https://img.shields.io/github/issues/KaHIP/HeiConnect)](https://github.com/KaHIP/HeiConnect/issues)
[![Last Commit](https://img.shields.io/github/last-commit/KaHIP/HeiConnect)](https://github.com/KaHIP/HeiConnect/commits)
[![SEA 2024](https://img.shields.io/badge/SEA'24-10.4230/LIPIcs.SEA.2024.11-blue)](https://doi.org/10.4230/LIPIcs.SEA.2024.11)
[![arXiv](https://img.shields.io/badge/arXiv-2402.07753-b31b1b.svg)](https://arxiv.org/abs/2402.07753)
[![Heidelberg University](https://img.shields.io/badge/Heidelberg-University-c1002a)](https://www.uni-heidelberg.de)
=====

<p align="center">
  <img src="https://raw.githubusercontent.com/KaHIP/HeiConnect/main/logo/heiconnect-banner.png" alt="HeiConnect Logo" width="900"/>
</p>

**HeiConnect** is a library of algorithms for the weighted connectivity augmentation problem on undirected graphs. Part of the [KaHIP](https://github.com/KaHIP) organization.

| | |
|:--|:--|
| **What it solves** | Find a minimum-cost set of links that increases the edge connectivity of a graph by one |
| **Algorithms** | Greedy heuristic (GWC), MST-Connect, local search, ILP-based exact, LP-based approximations |
| **Interfaces** | CLI |
| **Requires** | C++17 compiler, CMake 3.14+, [Gurobi](https://www.gurobi.com/solutions/gurobi-optimizer) >= 11.0.2, [Boost](https://boost.org) >= 1.83, [LEMON](https://lemon.cs.elte.hu/trac/lemon) >= 1.3.1 |

## Quick Start

```bash
git clone --recursive https://github.com/KaHIP/HeiConnect.git && cd HeiConnect
./build.sh
```

Verbose logging can be disabled at compile time with `-DENABLE_VERBOSE_LOG=OFF`.

### Run

```bash
# Compute cactus graph using VieCut
./extern/VieCut/build/mincut network.graph cactus -t network.xml

# Greedy Weight-Coverage heuristic
./deploy/solver -g network.graph -c network.xml -a gwc -o augmented.graph

# MST-Connect with local search
./deploy/solver -g network.graph -c network.xml -a mst-connect-ls --depth 3 -o augmented.graph

# Exact ILP solver
./deploy/solver -g network.graph -c network.xml -a eilp --use-initial -o augmented.graph
```

---

## Command Line Usage

```
./deploy/solver --graph <graph> --cactus <cactus> --algorithm <algorithm> [options]
```

### Required Arguments

| Option | Description |
|:-------|:-----------|
| `--graph <path>` | Input graph in [METIS format](http://people.sc.fsu.edu/~jburkardt/data/metis_graph/metis_graph.html) |
| `--cactus <path>` | Cactus graph in [GraphML](http://graphml.graphdrawing.org/) format (created by [VieCut](https://github.com/KaHIP/VieCut)) |
| `--algorithm <alg>` | Algorithm to use (see table below) |

### Optional Arguments

| Option | Description |
|:-------|:-----------|
| `--verbose` | Enable verbose logging |
| `--links <path>` | Link file: one link per line (`source target weight`) |
| `--output <path>` | Output file for the augmented graph |
| `--output-links` | Output only the selected links, not the full augmented graph |

### Algorithms

| Algorithm | Description | Extra Parameters | Best for |
|:----------|:-----------|:-----------------|:---------|
| `gwc` | Greedy Weight-Coverage heuristic with dynamic cactus and bounds | `--sampling=<0,1>` | small link costs (generated) |
| `mst-connect` | MST-Connect algorithm | | large link costs |
| `mst-connect-ls` | Local search on MST-Connect solution | `--depth=<int>`, `--cache`, `--trees=<int>` | large link costs |
| `eilp` | Exact ILP solver | `--use-initial` | small link costs (real-world) |
| `apx2lp` | LP-based 2-approximation | | |
| `apx1ln2e` | Better-than-2 approximation | `--epsilon=<float>` (default 0.15) | |
| `apx1.5e` | Better-than-2 approximation | `--epsilon=<float>` (default 0.15) | |
| `smc` | SMC with dynamic graph data structure | | |
| `fsm` | FSM with dynamic graph data structure | | |
| `hbd` | HBD with dynamic graph data structure | | |

---

## Graph Formats

### Input graph (METIS)

The input graph and resulting augmented graph use the [METIS ASCII graph format](http://people.sc.fsu.edu/~jburkardt/data/metis_graph/metis_graph.html). The first line contains the number of vertices and edges, and each subsequent line lists the neighbors of that vertex (1-indexed):

```
% Example: 4 vertices, 4 edges
4 4
2 3
1 3 4
1 2
2
```

### Cactus graph (GraphML)

The cactus graph uses the XML-based [GraphML](http://graphml.graphdrawing.org/) format with custom attributes: `containedVertices` for vertices and `weight` for edges. Cactus graphs are produced by [VieCut](https://github.com/KaHIP/VieCut) (`mincut <graph> cactus -t <output.xml>`). A cactus graph cannot be represented in METIS format because each cactus vertex contains a set of original graph vertices.

### Link file

An optional link file specifies candidate augmentation links, one per line in the format `source target weight`:

```
1 5 3.2
2 7 1.8
3 6 4.0
```

If no link file is provided, all possible links are considered.

## Graph Instances

Generated instances from the paper are in `graphs/generated`. A subset of 10th DIMACS Implementation Challenge graphs is also included; the full set is available at [dimacs10.github.io](https://dimacs10.github.io/downloads.shtml).

---

## Requirements

- C++17 compiler (GCC 7+ or Clang 11+)
- CMake 3.14+
- [Gurobi Optimizer](https://www.gurobi.com/solutions/gurobi-optimizer) >= 11.0.2
- [Boost](https://boost.org) >= 1.83 (max-flow)
- [LEMON](https://lemon.cs.elte.hu/trac/lemon) >= 1.3.1 (max-weight matching)

---

## Related Projects

| Project | Description |
|:--------|:------------|
| [KaHIP](https://github.com/KaHIP/KaHIP) | Karlsruhe High Quality Graph Partitioning |
| [VieCut](https://github.com/KaHIP/VieCut) | Shared-memory parallel minimum cut algorithms |
| [HeiCut](https://github.com/KaHIP/HeiCut) | Exact minimum cuts in hypergraphs |

---

## Licence

HeiConnect is free software provided under the MIT License.

If you publish results using our algorithms, please cite:

> Fonseca, M., Grossmann, E., Joos, F., Moller, T. and Schulz, C. (2024). Engineering Weighted Connectivity Augmentation Algorithms. *22nd International Symposium on Experimental Algorithms (SEA 2024)*, LIPIcs 301, pp. 11:1--11:22.

```bibtex
@inproceedings{DBLP:conf/wea/FarajGJM024,
  author       = {Marcelo Fonseca Faraj and
                  Ernestine Gro{\ss}mann and
                  Felix Joos and
                  Thomas M{\"{o}}ller and
                  Christian Schulz},
  editor       = {Leo Liberti},
  title        = {Engineering Weighted Connectivity Augmentation Algorithms},
  booktitle    = {22nd International Symposium on Experimental Algorithms, {SEA} 2024,
                  July 23-26, 2024, Vienna, Austria},
  series       = {LIPIcs},
  volume       = {301},
  pages        = {11:1--11:22},
  publisher    = {Schloss Dagstuhl - Leibniz-Zentrum f{\"{u}}r Informatik},
  year         = {2024},
  url          = {https://doi.org/10.4230/LIPIcs.SEA.2024.11},
  doi          = {10.4230/LIPICS.SEA.2024.11},
  timestamp    = {Fri, 12 Jul 2024 15:29:30 +0200},
  biburl       = {https://dblp.org/rec/conf/wea/FarajGJM024.bib},
  bibsource    = {dblp computer science bibliography, https://dblp.org}
}
```
