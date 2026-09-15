# Protein Stability Prediction Service (stabiliNNator)

## Overview

The Protein Stability Prediction Service identifies positions in an existing protein structure where a targeted mutation is likely to increase thermal stability. It takes a 3D structure — not a sequence — and scores every residue with a graph neural network.

Two complementary analyses are available, and they can be run together in one job:

* **proliNNator** — Which positions would tolerate, or benefit from, substitution to proline? Scores all residues except cysteines.
* **disulfiNNate** — Which cysteine positions are likely to form a stabilizing disulfide bond? Scores cysteines (CYS / CYX).

Both models are Graph Attention Networks that treat the structure as a graph of residues and their spatial neighbors. Each returns a probability between 0 and 1 per residue, written into the B-factor column of an annotated PDB so that any structure viewer can color by it directly.

Proline substitution and disulfide engineering are two of the most widely used strategies for rigidifying a protein — for example, stabilizing a viral glycoprotein in its prefusion conformation for vaccine design. This service ranks candidate positions so that wet-lab effort can be spent on the most promising handful.

## See also

* [Protein Stability Prediction Service](https://dxkb.org/app/StabiliNNator)
* [Protein Structure Prediction Service](https://dxkb.org/app/PredictStructure) — generates the structures this service scores

## PDB Selection

The protein selection option allows you to select a protein via:

* **Precomputed Structures (PDBB):** When provided a [Protein Data Bank](https://www.rcsb.org) ID, the service will download the PDB directly from the Protein Data Bank and use that structure.
* **Precomputed Structures (BVBRC):** Select a structure already held in the dxkb structure database. Use *Preview PDB* to confirm the structure before submitting.
* **Upload a PDB File from Workspace:** This option allows the use of a PDB file that has been uploaded to the user workspace. In the event the PDB file cannot be seen while selecting a PDB file, please ensure that the file is specified as a *pdb* type (or use the *show all files and folders* option while selecting).

The structure must contain standard amino acids with CA atoms — the graph is built from residue positions, so a backbone trace is the minimum required. Two input characteristics are handled automatically:

* **NMR / multi-model ensembles.** Only the first model is scored. Without this, a 38-model ensemble of a 20-residue peptide would be read as 760 concatenated residues and the scores would be meaningless.
* **Multiple chains.** All chains present are scored; results identify each residue by chain, position, and insertion code.

If you do not already have a structure, run the Protein Structure Prediction Service first and use its top-ranked PDB as the input here.

## Parameters

* **Analysis Type:** Which of the two models to run.
  * *Both* (default) — Runs proliNNator and disulfiNNate, producing both annotated PDBs and both ranked summaries. This is the common case; the two analyses are independent and together take only a few seconds longer than either alone.
  * *Proline* — Runs proliNNator only, producing the proline outputs.
  * *Disulfide* — Runs disulfiNNate only, producing the disulfide outputs.

Three further parameters are accepted by the service but deliberately not exposed on the submission form; they are available through the API:

* **accelerator** (default *cpu*) — *cpu* or *gpu*. CPU is recommended and is normally faster: the models are small (14–22 KB), so CUDA initialization overhead exceeds any compute savings.
* **hidden_dim** (default *32*) — Network hidden dimension. Both shipped models were trained with 32; change only when supplying custom-trained models.
* **dry_run** (default *false*) — Validates workspace access and input parsing, then stops without running predictions.

## Output

This selection specifies where the output of the service will be saved to the workspace. A workspace folder must be selected, and the output will be saved with the output name. The full path of the output would be *output folder* + *output name*. Give each run its own output name; two jobs sharing a name write into the same folder.

## Service Output

`<input>` below is the basename of the input structure, so scoring *crambin.pdb* yields *crambin_proline.pdb* and so on. Files for an analysis you did not request are simply absent.

* **stabilinnator_report.html** — Interactive HTML report; start here. It is self-contained, so no network access is needed to view it and it can be downloaded and shared as a single file. Select it in the workspace and use the REPORT action (the eye icon) or View. It contains the input structure and the models used with the exact commands run, an interactive 3D viewer colored by predicted probability, ranked tables of candidate positions, a per-residue stability track along the sequence, geometrically detected existing disulfide bonds, and links to the sibling data files.
* **`<input>`_proline.pdb** — Structure with proline probability in the B-factor column.
* **`<input>`_disulfide.pdb** — Structure with disulfide probability in the B-factor column.
* **`<input>`_proline_summary.tsv** — Full ranking, highest probability first.
* **`<input>`_disulfide_summary.tsv** — Cysteines only, ranked.
* **`<input>`_summary.json** — Combined machine-readable summary, capped at the top 25 sites per analysis. The TSVs keep the full ranking.

**Interpreting the scores.** The value in the B-factor column is a probability from 0 to 1; higher means a more favorable predicted site. These are rankings, not free-energy predictions: the score orders candidates against each other, and the intended use is to select the top few positions for experimental testing. Two caveats when reading the outputs directly:

* In the proline output, positions that are already proline typically score high — the model recognizes a proline-compatible environment. The TSV flags these with *already PRO* in its note column, since they are not actionable substitutions.
* In the disulfide output, every residue carries a B-factor, but only cysteines are biologically meaningful for disulfide formation. The ranked TSV filters to CYS/CYX for this reason; the annotated PDB does not.

**The ranked summaries.** Each TSV is ordered by probability, highest first, with these columns:

* *rank* — Position in the ranking, starting at 1.
* *chain* — Chain identifier (*-* if blank).
* *pos* — Residue number, with insertion code appended when present.
* *residue* — Three-letter residue name.
* *probability* — Predicted probability, 0–1, two decimal places.
* *note* — Proline analysis only; *already PRO* where applicable.

**Visualizing by score.** Because probabilities live in the B-factor column, any structure viewer can color by them. In the dxkb viewer, open an annotated PDB and color by B-factor: the highest-scoring positions stand out directly on the structure. The same file loads unmodified in PyMOL (*spectrum b*), ChimeraX, or Mol*.

Inference itself is sub-second for typical proteins; a complete *Both* run on a small protein finishes in roughly 12–16 seconds end to end. Scaling is mild with size — disulfiNNate's edge computation grows with the square of residue count, so an 8,000-residue complex takes on the order of 20 seconds rather than 1.

## References

* stabiliNNator source: [github.com/schoederlab/stabiliNNator](https://github.com/schoederlab/stabiliNNator)
* Veličković P et al. Graph Attention Networks. ICLR (2018). [arXiv:1710.10903](https://arxiv.org/abs/1710.10903)
