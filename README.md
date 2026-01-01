# PC Algorithm

A Python implementation of the PC (Peter-Clark) algorithm for causal discovery and learning the structure of Bayesian networks from observational data.

## Overview

The PC algorithm is a constraint-based method for learning causal structure from data. It works by performing conditional independence tests to construct a partially directed acyclic graph (PDAG) or completed partially directed acyclic graph (CPDAG) that represents the Markov equivalence class of the underlying causal structure.

This implementation provides:
- Skeleton estimation through conditional independence testing
- CPDAG estimation using orientation rules
- Support for binary and discrete data via the GSQ test framework

## Algorithm

The PC algorithm operates in two main phases:

### Phase 1: Skeleton Estimation

The algorithm starts with a complete undirected graph and iteratively removes edges based on conditional independence tests:

1. Begin with all nodes connected
2. Test independence between adjacent nodes conditioned on subsets of their neighbors
3. Remove edges when independence is detected
4. Incrementally increase the size of conditioning sets
5. Record separation sets for each removed edge

### Phase 2: Edge Orientation

After obtaining the skeleton, the algorithm orients edges using a series of rules:

1. **V-structures**: Orient edges into colliders (X → Z ← Y) when X and Y are not adjacent and Z is not in their separation set
2. **Rule 1**: Orient X-Y into X→Y when there exists X'→X and X' is not adjacent to Y
3. **Rule 2**: Orient X-Y into X→Y when there is a chain X→Z→Y
4. **Rule 3**: Orient X-Y into X→Y when there are two chains X-Z→Y and X-W→Y where Z and W are not adjacent

The result is a CPDAG representing the equivalence class of DAGs consistent with the data.

## Installation

### Prerequisites

- Python 3.7+
- NumPy
- NetworkX
- GSQ (for conditional independence testing)

### Setup

```bash
git clone https://github.com/lpossner/pc_algorithm.git
cd pc_algorithm
pip install numpy networkx gsq
```

## Usage

### Basic Example

```python
import numpy as np
from pc_algorithm import estimate_skeleton, estimate_cpdag
from gsq.ci_tests import ci_test_bin

# Load or generate your data (N x D array, where N is samples and D is variables)
data = np.random.randint(0, 2, size=(5000, 5))

# Estimate the skeleton
graph, separation_sets = estimate_skeleton(
    data=data,
    independence_test=ci_test_bin,
    significance_level=0.01
)

# Estimate the CPDAG
cpdag = estimate_cpdag(
    skeleton_graph=graph,
    separation_sets_lst=separation_sets
)

# The result is a NetworkX DiGraph
print(f"Nodes: {cpdag.nodes()}")
print(f"Edges: {cpdag.edges()}")
```

### Working with Different Data Types

For binary data:
```python
from gsq.ci_tests import ci_test_bin

graph, sep_sets = estimate_skeleton(
    data=binary_data,
    independence_test=ci_test_bin,
    significance_level=0.01
)
```

For discrete data:
```python
from gsq.ci_tests import ci_test_dis

graph, sep_sets = estimate_skeleton(
    data=discrete_data,
    independence_test=ci_test_dis,
    significance_level=0.05
)
```

## API Reference

### `estimate_skeleton(data, independence_test, significance_level)`

Estimates the skeleton of the causal graph.

**Parameters:**
- `data` (numpy.ndarray): Data matrix of shape (n_samples, n_variables)
- `independence_test` (callable): Function that takes (data, node_i, node_j, conditioning_set) and returns a p-value
- `significance_level` (float): Significance level for independence tests (e.g., 0.01, 0.05)

**Returns:**
- `graph` (networkx.Graph): Undirected skeleton graph
- `separation_sets_lst` (list): 2D list of separation sets for each pair of nodes

### `estimate_cpdag(skeleton_graph, separation_sets_lst)`

Estimates the CPDAG from a skeleton by orienting edges.

**Parameters:**
- `skeleton_graph` (networkx.Graph): Undirected skeleton graph from `estimate_skeleton`
- `separation_sets_lst` (list): Separation sets from `estimate_skeleton`

**Returns:**
- `dag` (networkx.DiGraph): Directed graph representing the CPDAG

## Example Output

The algorithm identifies causal relationships in the data. For instance, with the test data included in the repository:

```
Input: 5000 samples, 5 variables (binary)
Output CPDAG edges: [(0, 1), (2, 3), (3, 2), (3, 1), (2, 4), (4, 2), (4, 1)]
```

Where bidirectional edges (e.g., 2↔3, 2↔4) indicate that the direction cannot be determined from the data alone, while directed edges (e.g., 0→1) indicate a determined causal direction.

## Theory and Background

The PC algorithm is named after its inventors, Peter Spirtes and Clark Glymour. It's based on the following key assumptions:

1. **Causal Markov Condition**: Each variable is independent of its non-descendants given its parents
2. **Faithfulness**: The independence relationships in the data reflect exactly those implied by the causal structure
3. **Causal Sufficiency**: All common causes of measured variables are also measured

Under these assumptions, the PC algorithm is asymptotically correct—it converges to the true CPDAG as sample size approaches infinity.

## Testing

The implementation includes a basic test case that verifies the algorithm produces the expected output:

```python
# Run the main script
python pc_algorithm.py
```

The assertion at the end confirms that the estimated graph matches the expected structure.

## Limitations

- Requires large sample sizes for reliable results
- Sensitive to the choice of significance level
- Assumes causal sufficiency (no hidden confounders)
- Computational complexity increases with the number of variables
- May produce different results with different variable orderings in finite samples

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## References

1. Spirtes, P., Glymour, C., & Scheines, R. (2000). *Causation, Prediction, and Search*. MIT Press.
2. Colombo, D., & Maathuis, M. H. (2014). Order-independent constraint-based causal structure learning. *Journal of Machine Learning Research*, 15(1), 3741-3782.
3. Kalisch, M., & Bühlmann, P. (2007). Estimating high-dimensional directed acyclic graphs with the PC-algorithm. *Journal of Machine Learning Research*, 8, 613-636.

## Citation

If you use this implementation in your research, please cite:

```bibtex
@software{possner2024pc,
  author = {Possner, Lucas},
  title = {PC Algorithm: Python Implementation for Causal Discovery},
  year = {2024},
  url = {https://github.com/lpossner/pc_algorithm}
}
```

## Contact

Lucas Possner - [@lpossner](https://github.com/lpossner)

Project Link: [https://github.com/lpossner/pc_algorithm](https://github.com/lpossner/pc_algorithm)
