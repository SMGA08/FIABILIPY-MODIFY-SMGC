# Change Log

## Background
`fiabilipym` modernizes the original `fiabilipy` codebase for Python 3 while keeping the same reliability/availability mathematics and system-modeling API. As usage expanded across CI pipelines and mixed developer environments, Graphviz-based drawing became the main source of setup friction.

## Motivation
### Why Graphviz / pygraphviz was removed
The project removed the hard Graphviz/pygraphviz dependency from the default workflow for four practical reasons:

1. **Installation fragility**
   - `pygraphviz` frequently requires native Graphviz binaries and headers.
   - Install success depends on system-level packages, compiler toolchains, and PATH configuration.
   - This caused recurring build/install failures in lightweight or containerized environments.

2. **Cross-platform reproducibility issues**
   - Rendering and installation behavior varied across Linux/macOS/Windows.
   - Windows and conda-based setups were especially inconsistent due to binary compatibility and linking differences.
   - Teams saw “works on my machine” drift in diagram/test pipelines.

3. **Overhead relative to project needs**
   - For reliability block diagrams in this project, full Graphviz integration was more operational overhead than functional value.
   - The dependency chain increased onboarding and CI maintenance costs.

4. **Matplotlib-first replacement**
   - Diagram generation now emphasizes a lightweight Matplotlib block-style approach.
   - This is easier to run in CI/headless contexts and aligns with existing plotting dependencies.

## Detailed changes
### Dependency and packaging updates
- Removed `pygraphviz` from core dependencies in `pyproject.toml`.
- Added optional test dependencies:
  - `[project.optional-dependencies]`
  - `test = ["pytest>=8.0"]`

### Documentation updates
- `README.md` now explains Matplotlib-centric visualization usage.
- Added a minimal Matplotlib block diagram example suitable for local and CI use.
- Updated test instructions to include optional test extras and both pytest/unittest execution styles.

### Test strategy updates
- Graphviz-specific assertions and dependency expectations were removed from the test strategy.
- Draw-related tests now focus on headless-safe execution of plotting paths.
- The updated strategy expects draw methods to be verifiable in tests (including return-value checks for Matplotlib `Axes` where applicable).
- Matplotlib is configured to use the `"Agg"` backend in test contexts for CI/headless compatibility.

### Relevant project areas touched
This Graphviz-to-Matplotlib transition impacts or is reflected across:
- Core drawing surfaces and related docs (notably `system.py`, `markov.py`, and `README.md`).
- Tests under `tests/` that exercise draw paths.
- Packaging/dependency metadata (`pyproject.toml`, generated metadata files).
- Demo material/notebooks that present diagram generation workflows.

## How to test / run examples
### Install
```bash
pip install -e ".[test]"
```

### Run tests (pytest)
```bash
pytest -q
```

### Run tests (unittest discovery)
```bash
python -m unittest discover -s tests
```

### Minimal Matplotlib block-style example
```python
import matplotlib
matplotlib.use("Agg")

import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(5, 2))
ax.plot([0.1, 0.9], [0.5, 0.5], color="black", linewidth=1.5)
ax.text(0.08, 0.5, "E", va="center", ha="right")
ax.text(0.92, 0.5, "S", va="center", ha="left")
ax.add_patch(plt.Rectangle((0.4, 0.4), 0.2, 0.2, fill=False, linewidth=1.5))
ax.text(0.5, 0.5, "C1", va="center", ha="center")
ax.set_axis_off()
fig.tight_layout()
fig.savefig("minimal_block_diagram.png", dpi=150)
```

### Run the existing `System.draw()` flow in headless mode
```python
import matplotlib
matplotlib.use("Agg")

import matplotlib.pyplot as plt
from fiabilipym import Component, System

motor = Component("M", 1e-4, 3e-2)
power = Component("P", 1e-6, 2e-4)

S = System()
S["E"] = [power]
S[power] = [motor]
S[motor] = "S"

S.draw()
plt.tight_layout()
plt.savefig("block_diagram.png", dpi=150)
```
