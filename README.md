# GeoDistNet

GeoDistNet is an open-source research toolbox for generating GIS-informed synthetic distribution feeders and exporting them for power-flow analysis.

[![Python](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-research--toolbox-orange.svg)]()
[![GIS](https://img.shields.io/badge/GIS-OpenStreetMap-009e8d.svg)](https://www.openstreetmap.org/)
[![Power Flow](https://img.shields.io/badge/power--flow-pandapower-009e8d.svg)](https://pandapower.readthedocs.io/)
[![Optimization](https://img.shields.io/badge/optimization-MIP-6f42c1.svg)]()

## Overview

GeoDistNet turns publicly available GIS and road-network data into radial,
source-connected, simulation-ready synthetic distribution feeders. Starting
from GeoJSON line geometry (for example, road segments exported from
OpenStreetMap), it builds a candidate graph, synthesizes a constrained radial
feeder, assigns representative electrical parameters and loads, and exports the
result to [pandapower](https://pandapower.readthedocs.io/) for steady-state
power-flow validation.

The toolbox is organized as a set of modular library functions under
`src/network/` and a small collection of runnable scripts under `experiments/`.
It is intended for reproducible distribution-network modeling and experimentation,
not as a replacement for utility planning tools. Generated networks are
representative synthetic feeders derived from public data, not utility-calibrated
replicas.

## What GeoDistNet does

- builds candidate graphs from public geographic data (GeoJSON / OpenStreetMap road networks);
- synthesizes radial, source-connected feeder topologies using a constrained MIP formulation;
- assigns representative line impedances and allocates household loads across buses;
- exports synthetic feeders to pandapower as `buses.csv`, `lines.csv`, and `feeder_params.yaml`;
- validates generated feeders through steady-state power flow and loading scenarios.

## Workflow

The toolbox follows a single, reusable pipeline. Each stage maps to modular
functions in `src/network/` and can be driven end to end by the scripts in
`experiments/`.

```text
GIS / OSM data
  → candidate graph extraction
  → constrained radial feeder synthesis
  → electrical parameter assignment
  → load allocation
  → pandapower export
  → load-flow validation
```

A lightweight synthetic generator is also provided for cases where no GIS input
is required, producing a radial feeder directly from geometric and electrical
parameters.

## Key features

- **Reusable pipeline.** A consistent GIS-to-pandapower workflow that can be
  reused across study areas and parameter sets.
- **Modular scripts.** Each stage is exposed as an importable function in
  `src/network/`, with thin command-line wrappers in `experiments/`.
- **Public data input.** Works from publicly available GIS / road-network data;
  no confidential utility feeder data is required.
- **Constrained radial synthesis.** Builds radial, source-connected feeders with
  a MIP formulation that bounds downstream node counts per edge; an MST baseline
  is available for comparison.
- **Simulation-ready export.** Exports feeders directly to pandapower for
  steady-state analysis.
- **Reproducible modeling.** Deterministic seeding for the synthetic generator
  and small bundled fixtures for quick, repeatable runs.

## Installation

GeoDistNet targets Python 3.9+ (tested on 3.11).

```bash
conda create -n geodistnet python=3.11
conda activate geodistnet
pip install -r requirements.txt
```

> Note: if `requirements.txt` is not yet present in your checkout, the core
> dependencies can be installed directly:
>
> ```bash
> pip install numpy pandas scipy networkx matplotlib pyproj pyomo pandapower pyyaml
> ```
>
> Optional extras: `geopandas` and `contextily` (basemap plotting), and
> `requests` (fetching OpenStreetMap data via the Overpass API).

The constrained synthesis step requires an LP/MIP solver. GeoDistNet uses Pyomo
and attempts solvers in the following order:

```text
gurobi → highs → cbc → glpk
```

See [Notes on solvers and reproducibility](#notes-on-solvers-and-reproducibility).

## Quick start

### Lightweight demo

Generate a synthetic radial feeder (no GIS data, no external solver needed) and export it:

```bash
python experiments/build_synthetic_network.py
```

Run the GIS demo (single GeoJSON → MIP → pandapower) on a bundled fixture :

```bash
python experiments/run_gis_demo.py
```

### Full GIS-based workflow

Optionally fetch a real-world road network from OpenStreetMap (Overpass API)
and write it to GeoJSON:

```bash
python experiments/build_real_world_case.py
```

Run the full pipeline (GIS → MIP feeder synthesis → loading-scenario validation):

```bash
python experiments/run_path_c_pipeline.py
```

### Additional workflows

Compare the MST baseline against the constrained MIP feeder:

```bash
python experiments/run_mst_vs_mip.py
```

Run loading scenarios on already-exported feeder files:

```bash
python experiments/applications/run_loading_scenarios.py \
  --data-dir data/network \
  --out-dir data/results
```

Most scripts accept command-line arguments (for example, root coordinates,
solver choice, and load settings). Run a script with `--help` to list its
options.

### Inputs

The GIS path expects GeoJSON `LineString` features in WGS-84. Minimal fixtures
are provided in `data/examples/`:

- `simple_lv_feeder.geojson`
- `lv_feeder_32bus.geojson`
- `osm_bethnal_green.geojson`
- `osm_richmond_melbourne.geojson`

### Outputs

Each generated feeder is exported as `buses.csv`, `lines.csv`, and
`feeder_params.yaml`. Loading-scenario validation additionally writes summary
tables and figures (for example, voltage profile and branch loading).

### Tests

```bash
conda run -n geodistnet pytest tests/ -v
```

## Example outputs

A sample GIS-derived feeder over a basemap:

<img width="400" height="400" alt="GIS-derived feeder example" src="https://github.com/user-attachments/assets/2483ece0-ac41-450a-bf02-89ffa520cdce" />

The following figures are planned for the documentation gallery and are **not yet
committed**. Once added under `docs/figures/`, they can be embedded here:

- [ ] `docs/figures/workflow.png` — end-to-end pipeline diagram
- [ ] `docs/figures/example_feeder.png` — synthesized radial feeder
- [ ] `docs/figures/voltage_profile.png` — voltage profile from load-flow validation

## Notes on solvers and reproducibility

- **Solver.** Gurobi is recommended for the MIP-based synthesis step. When it is
  unavailable, GeoDistNet falls back through `highs`, `cbc`, and `glpk`. Open-source
  solvers can solve the same models, but runtime and solution quality may differ,
  particularly on larger candidate graphs.
- **Data.** The toolbox uses public GIS data and does not require confidential
  utility feeder data.
- **Synthetic nature.** Generated networks are representative synthetic feeders,
  not utility-calibrated replicas, and the assigned parameters are intended for
  modeling and experimentation.
- **Reproducibility.** The synthetic generator supports a fixed random seed, and
  the bundled fixtures allow short, repeatable runs without external downloads.

## License

Released under the MIT License. See [LICENSE](LICENSE).

## Citation / acknowledgement

If you use GeoDistNet in academic work, please cite this repository. A
`CITATION.cff` file is planned to provide structured citation metadata.

This toolbox builds on open data and open-source tools, including
[OpenStreetMap](https://www.openstreetmap.org/) contributors for geographic data
and [pandapower](https://pandapower.readthedocs.io/) for power-flow analysis.
