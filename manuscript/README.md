# Manuscript — ONNX-EdgeIDS Paper (Springer LNCS)

Self-contained English draft: `llncs.cls`, `splncs04.bst`, and the XeLaTeX font setup are vendored under `vendor/`, so this compiles without the Thesis_IDS monorepo or any Springer template checkout.

## Compile

```bash
cd manuscript
./compile.sh
```

Compiles cleanly as of this draft (14 pages, 0 undefined citations/references — only cosmetic under/overfull-hbox warnings in the bibliography).

## TODO before submission

- [ ] Pick a venue and deadline, update `../README.md` and the title page
- [x] Author names and affiliations (copied from the SOICT companion in Thesis_IDS — re-confirm order/roles before submission; note the software artifact itself, `onnx-edge-ids`, is single-authored by Thai Nguyen Vu per its `CITATION.cff` — confirm this paper's author list is what's intended)
- [x] No PySpark/Spark content anywhere in `main.tex`/`references.bib`
- [x] Repo is self-contained: no `\input`/`\bibliography` paths outside this repo (vendored `llncs.cls`, `splncs04.bst`, font setup, and the two bib entries `main.tex` needed from the monorepo's shared `related_work.bib`)
- [x] Boundary-directed parity experiment (`../code/experiments/exp2_boundary_parity.py`): 287,223 probes, float32 0 flips, float64 328 flips vs 15 on full replay
- [x] Block-lockstep traversal measured on real flows, including the batch size where it loses (`exp1`); shipped in `onnx-edge-ids` at `894238a`
- [x] Exact early exit measured on gate-forwarded traffic (`exp3`)
- [ ] Re-run `exp1` and `exp3` on the Jetson Cortex-A78AE boards and merge into Table `tab:engines`
- [ ] Re-run `exp2` against any other ONNX Runtime provider that will be deployed (CUDA/TensorRT): a different provider is a different runtime
- [ ] Decide whether the boundary prober should move into `onnx-edge-ids` as part of its release gate
- [ ] Run the wired-link end-to-end benchmark and fill the remaining `\ph{...}`
- [ ] Repeat the steady-state/cold-start/parity measurements across multiple runs (the software's own acceptance criteria call for this; current numbers are a single run)
- [ ] Verify target venue's page limit/template requirements once chosen
- [ ] Decide whether/how to cross-reference the parallel Software Impacts submission for the same software once that one is finalized, so the two don't duplicate claims
