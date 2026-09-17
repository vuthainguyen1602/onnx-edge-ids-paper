# Experiment code

Self-contained code for the measurements in the manuscript. It does not import the `onnx-edge-ids` package: the forest traversal, the early-exit rule and the boundary prober live here so the paper's numbers can be regenerated from this repo alone.

```
code/
  edgeids/forest.py      per-tree loop, block-lockstep traversal, exact early exit
  edgeids/data.py        labelled test split loader, deployed anomaly gate
  experiments/           one script per experiment, each writes results/<name>.json
  artifacts/             the served ONNX and NumPy forests, feature order, gate artifacts
  results/               JSON written by the last run (the numbers quoted in the paper)
  tests/                 run on the shipped artifact alone, no dataset needed
```

## Setup

```bash
python -m venv venv && source venv/bin/activate
pip install -r code/requirements.txt
python -m pytest code/tests -q
```

## Data

The experiments read the labelled CICIDS2017 **test split** as a Parquet directory with the 29 deployed feature columns (`artifacts/feature_columns.json`) plus `label` and `label_binary`. The dataset is not redistributed here; it comes from the Canadian Institute for Cybersecurity (Sharafaldin et al., ICISSP 2018). The split used for the paper has 447,275 rows (15.0% attacks).

## Experiments

```bash
DATA=/path/to/test_data.parquet
python code/experiments/exp1_lockstep_benchmark.py --data $DATA   # Table: per-tree vs lockstep
python code/experiments/exp2_boundary_parity.py    --data $DATA   # Table: boundary-directed parity
python code/experiments/exp3_exact_early_exit.py   --data $DATA   # Table: exact early exit
```

| Script | Question | Headline result (`results/*.json`) |
|--------|----------|-----------------------------------|
| `exp1` | Does advancing trees in lockstep beat one Python loop per tree, and where does it stop winning? | 71x at batch 1, 11x at 20, 1.4x at 500, 0.9x at 2000 |
| `exp2` | Does replaying random rows find a runtime that disagrees with ONNX Runtime at split thresholds? | float64 traversal: 328 label flips on 287,223 boundary probes, 15 on 447,275 replayed rows; float32: 0 and 0 |
| `exp3` | How many of the 200 trees does traffic need if labels must stay identical? | 104.1 on gate-forwarded flows, 2.0x, 50,548/50,548 labels identical |

Throughput numbers were taken on an Apple M3 (ARM64); they have not been re-measured on the Jetson boards. Label-agreement and disagreement counts do not depend on the machine.

`anomaly_*.pkl` were pickled with an older scikit-learn; loading them under a newer one prints a version warning.
