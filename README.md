# ONNX-EdgeIDS Paper

Full systems-paper (Springer LNCS) treatment of the [`onnx-edge-ids`](https://github.com/vuthainguyen1602/onnx-edge-ids) software: a framework-agnostic ONNX/NumPy serving toolkit for a two-stage Kafka edge intrusion detection pipeline on Jetson Orin Nano Super.

This repo was split out of the `Thesis_IDS` monorepo (was `papers/onnx2026/`) so it can be versioned and reviewed independently of the core thesis papers (SOICT/FAIR), which describe a different, earlier deployment and stay in that repo.

**Venue:** TBD — draft, complementary in scope to a separate, shorter **Software Impacts** submission for the same software (see `notes/paper_pitch.md` and `submission_checklist.md` in the [software repo](https://github.com/vuthainguyen1602/onnx-edge-ids)). This paper is the fuller architecture/related-work/discussion treatment; the two are meant to be read independently, not merged.

## Research question

A detector served by a primary runtime (ONNX Runtime) and a fallback runtime (NumPy) promises the same verdict from both. How do you check that promise, what does the fallback cost at streaming batch sizes, and how much of the forest does real traffic need if verdicts may not change?

Findings, all regenerable from `code/`:

- **Boundary-directed parity testing.** One probe per split node, on the threshold and one float32 step either side (287,223 probes, 99.9% reach their node). A float64 NumPy traversal flips 328 labels against ONNX Runtime on the probes but only 15 on the whole 447,275-row labelled test split, so a 2,000-row replay gate misses it 93.5% of the time. The deployed float32 traversal flips none.
- **Why labels flip.** The forest's 95,741 splits share 815 distinct (feature, threshold) pairs, so one boundary value moves many trees at once; the same fact makes an exhaustive gate cheap.
- **Block-lockstep traversal.** 71x at batch 1, 11x at 20, 1.4x at 500, and 0.9x at 2,000 (where the old per-tree loop still wins), on real flows.
- **Exact early exit.** Gate-forwarded traffic needs 104.1 of 200 trees on average with labels provably identical to the full forest: 2.0x, the rule's ceiling.
- **Distribution shift.** This work does not make the detector robust to drift. Frozen and run on CSE-CIC-IDS2018, classifier F1 falls 0.98 to 0.24 and gate attack recall 0.72 to 0.01. Runtime parity and early-exit exactness hold on all 477,771 shifted flows, and the share of flows early exit cannot settle rises 4.8% to 28.2% without labels.

## Title

*Two Runtimes, One Verdict: Boundary-Verified ONNX/NumPy Serving for Streaming Intrusion Detection at the Edge*

## Code

**Experiment code lives here**, in [`code/`](code/): the traversals, the early-exit rule, the boundary prober, the served artifacts, the result files quoted in the paper, and tests that run without the dataset. See [`code/README.md`](code/README.md).

**The serving software** (Kafka gate and classifier nodes, exporter, release gate) stays in its own repo, [`vuthainguyen1602/onnx-edge-ids`](https://github.com/vuthainguyen1602/onnx-edge-ids) (Zenodo concept DOI `10.5281/zenodo.22731927`). The block-lockstep traversal measured here is the one shipped there as of commit [`894238a`](https://github.com/vuthainguyen1602/onnx-edge-ids/commit/894238a); the on-board figures in the paper predate it and describe the per-tree traversal.

| Component | Path (in `onnx-edge-ids`) |
|-----------|------|
| Swappable engine contract | `src/onnx_edge_ids/inference_engine.py` |
| Feature assembly | `src/onnx_edge_ids/feature_matrix.py` |
| ONNX backend | `src/onnx_edge_ids/onnx_engine.py` |
| NumPy backend (block-lockstep traversal) | `src/onnx_edge_ids/numpy_engine.py` |
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
git checkout 894238a   # the traversal measured in this draft
ONNX_MODEL=artifacts/ids_rf.onnx FEATURES_JSON=artifacts/feature_columns.json ./scripts/export_artifacts.sh
python scripts/validate_parity.py --csv <path to a CICIDS2017-derived replay CSV> --rows 2000 \
  --report-json results/parity_report.json
```

**Step 2 — two-Jetson deployment, one classifier-node run per backend and per batch size (fills Table `tab:engines`, batch-size sensitivity in Sect. "Results"):**

Follow `docs/deploy_2jetson.md` in the software repo: gate on Jetson #1, `classifier_node.py --engine onnx|numpy --batch-size 1|20|500 --metrics-csv ...` on Jetson #2.

**Step 3 — regenerate the boundary-parity, traversal and early-exit tables** with `code/experiments/` (see `code/README.md`), then re-run `exp1` and `exp3` on the Jetson boards: their throughput numbers were taken on an ARM64 development machine.

**Step 4 — wired-link end-to-end run (the pending measurement):**

Repeat Step 2 over a wired inter-board link instead of Wi-Fi, and report producer-timestamp-to-verdict latency/throughput, not just in-process classifier cost.

## Metrics to report

- Classifier-node throughput (rows/s) and p95 (ms/row), per backend and per batch size
- Cold-start time to first verdict, per backend
- RSS (MB) and total-board power (W, `tegrastats`, not idle-subtracted)
- Label agreement / max confidence deviation between backends (parity gate)
- Artifact size (ONNX vs. NumPy)
- End-to-end (producer-to-verdict) throughput/latency over a **wired** link — pending
- Boundary-probe and replay disagreement counts per runtime (`exp2`)
- Detector, gate, parity and early-exit behaviour on shifted traffic (`exp4`)
- Block-lockstep and exact-early-exit throughput, re-measured on Jetson hardware — pending

## Manuscript

The English draft is in `manuscript/` (Springer LNCS template, XeLaTeX; vendored `llncs.cls`/`splncs04.bst`/font setup under `manuscript/vendor/` so this repo has no dependency on the Thesis_IDS monorepo). Compile with:

```bash
cd manuscript
./compile.sh
```

Compiles cleanly (15 pages, 0 undefined citations/references). On-board numbers come from the software's own validation run; the boundary-parity, traversal and early-exit tables come from `code/results/*.json`. The remaining `\ph{...}` markers are the Jetson re-measurement of those two throughput tables and the wired-link latency.

## Cross-reference

- Software artifact and short-form Software Impacts submission for the same toolkit: [`vuthainguyen1602/onnx-edge-ids`](https://github.com/vuthainguyen1602/onnx-edge-ids)
- Prior distributed-deployment context (out of scope here) and the core thesis papers: [`vuthainguyen1602/Thesis_IDS`](https://github.com/vuthainguyen1602/Thesis_IDS) (`papers/soict2026/`, `papers/fair2026/`, `thesis/`)
