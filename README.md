# ONNX-EdgeIDS Paper

Full systems-paper (Springer LNCS) treatment of the [`onnx-edge-ids`](https://github.com/vuthainguyen1602/onnx-edge-ids) software: a framework-agnostic ONNX/NumPy serving toolkit for a two-stage Kafka edge intrusion detection pipeline on Jetson Orin Nano Super.

This repo was split out of the `Thesis_IDS` monorepo (was `papers/onnx2026/`) so it can be versioned and reviewed independently of the core thesis papers (SOICT/FAIR), which describe a different, earlier deployment and stay in that repo.

**Venue:** TBD — draft, complementary in scope to a separate, shorter **Software Impacts** submission for the same software (see `notes/paper_pitch.md` and `submission_checklist.md` in the [software repo](https://github.com/vuthainguyen1602/onnx-edge-ids)). This paper is the fuller architecture/related-work/discussion treatment; the two are meant to be read independently, not merged.

## Research question

Earlier deployment work found that an accurate classifier can still be served too slowly for an 8 GB edge board, if the serving stack is inherited unchanged from training. This paper presents the resulting software response: a two-runtime (ONNX Runtime / pure NumPy) serving toolkit built around a single ONNX artifact, with a framework-agnostic NumPy exporter and a release-gated parity validator, integrated into a two-stage Kafka gate-then-classifier pipeline — plus a measured optimization (not just a future-work citation) of the NumPy backend's tree-traversal loop.

## Title

*ONNX-EdgeIDS: A Framework-Agnostic ONNX/NumPy Serving Toolkit for Streaming Intrusion Detection on Jetson Orin Nano Super*

## Code: referenced, not duplicated

This repo holds the manuscript only. The software it describes lives and is versioned in its own repo, [`vuthainguyen1602/onnx-edge-ids`](https://github.com/vuthainguyen1602/onnx-edge-ids) (Zenodo concept DOI `10.5281/zenodo.22731927`); we reference it rather than vendor a source snapshot here, so there is exactly one place the code can drift from what the paper says.

**Pinned commit for this draft's measurements:** the throughput/latency/memory/power/parity numbers in the current manuscript, and the lockstep-traversal optimization in Sect. "A Lockstep Traversal Optimization", correspond to `onnx-edge-ids` commit [`59f2723`](https://github.com/vuthainguyen1602/onnx-edge-ids/commit/59f2723). If `main` has moved past that commit when you read this, check out that SHA to reproduce the exact numbers in the paper; re-measuring against a later `main` is also fine but should be noted as such.

| Component | Path (in `onnx-edge-ids`) |
|-----------|------|
| Swappable engine contract | `src/onnx_edge_ids/inference_engine.py` |
| Feature assembly | `src/onnx_edge_ids/feature_matrix.py` |
| ONNX backend | `src/onnx_edge_ids/onnx_engine.py` |
| NumPy backend (lockstep traversal, Sect. "A Lockstep Traversal Optimization") | `src/onnx_edge_ids/numpy_engine.py` |
| Framework-agnostic ONNX → NumPy exporter | `scripts/export_numpy.py` |
| Parity release gate | `scripts/validate_parity.py` |
| Jetson #1 gate node | `scripts/gate_node.py` |
| Jetson #2 classifier node | `scripts/classifier_node.py` |
| CICIDS2017-derived replay producer | `scripts/csv_producer.py` |
| Two-Jetson deployment guide | `docs/deploy_2jetson.md` |
| Real validation-run numbers used in this manuscript | `submission_checklist.md` |

## Reproduce

**Step 1 — export the NumPy fallback from an ONNX artifact, then validate parity (fills Table `tab:parity`):**

```bash
git clone https://github.com/vuthainguyen1602/onnx-edge-ids
cd onnx-edge-ids
git checkout 59f2723   # pin to the commit this draft's numbers correspond to
ONNX_MODEL=artifacts/ids_rf.onnx FEATURES_JSON=artifacts/feature_columns.json ./scripts/export_artifacts.sh
python scripts/validate_parity.py --csv <path to a CICIDS2017-derived replay CSV> --rows 2000 \
  --report-json results/parity_report.json
```

**Step 2 — two-Jetson deployment, one classifier-node run per backend and per batch size (fills Table `tab:engines`, batch-size sensitivity in Sect. "Results"):**

Follow `docs/deploy_2jetson.md` in the software repo: gate on Jetson #1, `classifier_node.py --engine onnx|numpy --batch-size 1|20|500 --metrics-csv ...` on Jetson #2.

**Step 3 — re-measure the lockstep traversal optimization (Sect. "A Lockstep Traversal Optimization") on the Jetson boards themselves**, rather than the ARM64 development machine it was first measured on; fold the result into Table `tab:engines`.

**Step 4 — wired-link end-to-end run (the pending measurement):**

Repeat Step 2 over a wired inter-board link instead of Wi-Fi, and report producer-timestamp-to-verdict latency/throughput, not just in-process classifier cost.

## Metrics to report

- Classifier-node throughput (rows/s) and p95 (ms/row), per backend and per batch size
- Cold-start time to first verdict, per backend
- RSS (MB) and total-board power (W, `tegrastats`, not idle-subtracted)
- Label agreement / max confidence deviation between backends (parity gate)
- Artifact size (ONNX vs. NumPy)
- End-to-end (producer-to-verdict) throughput/latency over a **wired** link — pending
- Lockstep-traversal speedup, re-measured on Jetson hardware — pending

## Manuscript

The English draft is in `manuscript/` (Springer LNCS template, XeLaTeX; vendored `llncs.cls`/`splncs04.bst`/font setup under `manuscript/vendor/` so this repo has no dependency on the Thesis_IDS monorepo). Compile with:

```bash
cd manuscript
./compile.sh
```

Compiles cleanly (12 pages, 0 undefined citations/references). The numbers in Sect. "Results" are real (from the software's own validation run and this paper's own measured lockstep-traversal benchmark); the remaining `\ph{...}` markers are the wired-link measurement and the Jetson re-measurement of the traversal optimization.

## Cross-reference

- Software artifact and short-form Software Impacts submission for the same toolkit: [`vuthainguyen1602/onnx-edge-ids`](https://github.com/vuthainguyen1602/onnx-edge-ids)
- Prior distributed-deployment context (out of scope here) and the core thesis papers: [`vuthainguyen1602/Thesis_IDS`](https://github.com/vuthainguyen1602/Thesis_IDS) (`papers/soict2026/`, `papers/fair2026/`, `thesis/`)
