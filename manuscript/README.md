# Manuscript — ONNX-EdgeIDS Paper (Springer LNCS)

Self-contained English draft: `llncs.cls`, `splncs04.bst`, and the XeLaTeX font setup are vendored under `vendor/`, so this compiles without the Thesis_IDS monorepo or any Springer template checkout.

## Compile

```bash
cd manuscript
./compile.sh
```

Compiles cleanly as of this draft (12 pages, 0 undefined citations/references — only cosmetic under/overfull-hbox warnings in the bibliography).

## TODO before submission

- [ ] Pick a venue and deadline, update `../README.md` and the title page
- [x] Author names and affiliations (copied from the SOICT companion in Thesis_IDS — re-confirm order/roles before submission; note the software artifact itself, `onnx-edge-ids`, is single-authored by Thai Nguyen Vu per its `CITATION.cff` — confirm this paper's author list is what's intended)
- [x] No PySpark/Spark content anywhere in `main.tex`/`references.bib`
- [x] Repo is self-contained: no `\input`/`\bibliography` paths outside this repo (vendored `llncs.cls`, `splncs04.bst`, font setup, and the two bib entries `main.tex` needed from the monorepo's shared `related_work.bib`)
- [x] Lockstep NumPy traversal optimization (Sect. "A Lockstep Traversal Optimization") implemented, correctness-verified (5,000/5,000 label agreement vs. the prior engine; `validate_parity.py` still passes), and benchmarked (up to 84x at batch 1) — pushed to `onnx-edge-ids` main at commit `59f2723` (see `../README.md`)
- [ ] Re-measure the lockstep traversal on the Jetson Cortex-A78AE boards (currently measured on ARM64 dev hardware only) and fold into Table `tab:engines`
- [ ] Run the wired-link end-to-end benchmark and fill the remaining `\ph{...}`
- [ ] Repeat the steady-state/cold-start/parity measurements across multiple runs (the software's own acceptance criteria call for this; current numbers are a single run)
- [ ] Verify target venue's page limit/template requirements once chosen
- [ ] Decide whether/how to cross-reference the parallel Software Impacts submission for the same software once that one is finalized, so the two don't duplicate claims
