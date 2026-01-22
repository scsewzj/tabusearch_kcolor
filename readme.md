# Tabu Search for k-Graph Coloring

## Overview

This project implements a Tabu Search metaheuristic algorithm to solve the k-Graph Coloring Problem. The k-Graph Coloring Problem is an NP-complete combinatorial optimization task where the goal is to assign one of k colors to each vertex in an undirected graph such that no two adjacent vertices share the same color (i.e., minimize conflicts to zero).

The algorithm is tested on randomly generated graphs using the Erdős–Rényi model. Experiments evaluate performance across parameters like graph size (n), edge probability (p), number of colors (k), and tabu tenure.

This is a course report implementation for "Algorithmic and Combinatorial Optimisation" at Université Paris-Saclay (M2 GENIOMHE).

## Key Features

- **Problem Formulation**: Reformulates graph coloring as minimizing conflicting edges.
- **Algorithm**: Tabu Search with aspiration criterion, efficient delta conflict calculation, and focus on conflicting vertices.
- **Graph Generation**: Uses NetworkX to create Erdős–Rényi graphs.
- **Experiments**: Analyzes scalability, density effects, color constraints, and tabu tenure impact.
- **Results**: Shows effective for sparse graphs; performance degrades with density and size.

## Requirements

- Python 3.6+
- NetworkX (`pip install networkx`)
- Random (standard library)

## Installation

1. Clone the repository:
   ```
   git clone https://github.com/scsewzj/tabusearch_kcolor
   cd tabusearch_kcolor
   ```

2. Install dependencies:
   ```
   pip install networkx
   ```

## Usage

The core implementation is in the `TabuGraphColoring` class in the [main jupyter notebook](/main.ipynb). Here's a basic example:

```python
import networkx as nx
import random

# Define the class (copy from the report's code listing)
class TabuGraphColoring:
    def __init__(self, G, k, tabu_tenure=7, max_iter=1000, mute=False):
        self.G = G
        self.V = list(G.nodes)
        self.k = k
        self.tabu_tenure = tabu_tenure
        self.max_iter = max_iter
        self.it = -1
        self.reached_max_iter = False
        self.mute = mute

        # Random initial coloring
        self.color = {v: random.randrange(k) for v in self.V}
        self.best_color = self.color.copy()
        self.best_conflicts = self.conflicts()

        # Tabu list: (v, color) -> expire_iter
        self.tabu_list = {}

    def conflicts(self, coloring=None):
        if coloring is None:
            coloring = self.color
        cnt = 0
        for u, v in self.G.edges:
            if coloring[u] == coloring[v]:
                cnt += 1
        return cnt

    def conflicting_vertices(self):
        bad = set()
        for u, v in self.G.edges:
            if self.color[u] == self.color[v]:
                bad.add(u)
                bad.add(v)
        return list(bad)

    def search(self):
        for it in range(self.max_iter):
            current_conflicts = self.conflicts()

            if current_conflicts == 0:
                if not self.mute:
                    print(f"Found proper coloring at iteration {it}")
                self.it = it
                return self.color

            best_move = None
            best_delta = float("inf")

            conflict_vertices = self.conflicting_vertices()
            random.shuffle(conflict_vertices)

            for v in conflict_vertices:
                old_color = self.color[v]
                for c in range(self.k):
                    if c == old_color:
                        continue

                    # Delta calculation: only neighbors of v
                    delta = 0
                    for u in self.G.neighbors(v):
                        if self.color[u] == old_color:
                            delta -= 1
                        if self.color[u] == c:
                            delta += 1

                    # Tabu check
                    tabu = (v, old_color) in self.tabu_list and self.tabu_list[(v, old_color)] > it

                    # Aspiration criterion
                    if tabu and current_conflicts + delta >= self.best_conflicts:
                        continue

                    if delta < best_delta:
                        best_delta = delta
                        best_move = (v, c, old_color)

            if best_move is None:
                continue

            v, new_color, old_color = best_move
            self.color[v] = new_color
            self.tabu_list[(v, old_color)] = it + self.tabu_tenure

            new_conflicts = current_conflicts + best_delta
            if new_conflicts < self.best_conflicts:
                self.best_conflicts = new_conflicts
                self.best_color = self.color.copy()

        if not self.mute:
            print("Reached max iterations")
        self.it = self.max_iter
        self.reached_max_iter = True
        return self.best_color

# Example usage
n = 50  # Number of vertices
p = 0.1  # Edge probability
k = 4  # Number of colors

G = nx.erdos_renyi_graph(n, p)
solver = TabuGraphColoring(G, k)
coloring = solver.search()
print(f"Final conflicts: {solver.best_conflicts}")
```

Run the script to solve a random graph instance.

## Experiments and Results

Experiments were conducted on Erdős–Rényi graphs:

- **Graph Size (n=20-100)**: Iterations follow an S-shaped curve, rising sharply around n=40. Conflicts grow like a power function beyond n=60.
- **Edge Probability (p=0.05-0.2)**: Similar to size effects, performance tied to average node degree.
- **Number of Colors (k=3-7)**: For k=3, often fails; success improves with higher k.
- **Tabu Tenure (3-20)**: Moderate values (e.g., 5-10) perform best; extremes lead to cycling or restriction.

For details, see the full report [PDF](/main.pdf).

## Author

- Zhouji WU , an M2 GENIOMHE student, at Université Paris-Saclay

## License

This project is for educational purposes. No specific license; feel free to use for non-commercial academic work.