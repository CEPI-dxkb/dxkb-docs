# FrustraMPNN Service

## Overview

This service runs [FrustraMPNN](https://github.com/RosettaCommons/frustraMPNN) on our servers. The FrustraMPNN tool takes a PDB file as input and predicts frustration given single point mutations for each protein in the given PDB file.

## See also

* [FrustraMPNN Service](https://dxkb.org/app/FrustraMPNN)

## PDB Selection

The protein selection option allows you to select a protein via:

* **Precomputed Structures:** When provided a [Protein Data Bank](https://www.rcsb.org) ID, the service will download the PDB directly from the Protein Data Bank and use that PDB in FrustraMPNN.
* **Upload a PDB File:** This option allows the use of a PDB file that has been uploaded to the user workspace. In the event the PDB file cannot be seen while selecting a PDB file, please ensure that the file is specified as a *pdb* type (or use the *show all files and folders* option while selecting).

## Parameters

There are two parameters that affect the output of the FrustraMPNN service:

* **Weights:** There are two sets of model weights available to choose from:
  * *fireprot* — Trained on the FireProt dataset, which contains natural protein mutation data and exhibits a diverse range of protein lengths and structural architectures.
  * *megascale* — Trained on the massive Megascale dataset, which includes over 700,000 experimental measurements from both natural and de novo-designed (artificial) protein domains.
* **Chains:** Specify which chains to compute frustration on. If left empty, will compute on all chains. For example, "A C" will predict on chains *A* and *C*.

## Output

This selection specifies where the output of the service will be saved to the workspace. A workspace folder must be selected, and the output will be saved with the output name. The full path of the output would be *output folder* + *output name*.

## Service Output

There are three outputs for the FrustraMPNN service:

* **frustrampnn.out.csv** — CSV file containing the following columns:
  * *(unnamed)* — Sequential index column starting at 0.
  * *frustration_pred* — Predicted frustration value.
  * *position* — Position of the mutation.
  * *wildtype* — The original amino acid at the given position.
  * *mutation* — The mutation that produces the frustration prediction.
  * *chain* — Which chain the position is on.
  * *pdb* — The PDB file used.
* **frustrampnn.out.plotly.heamap.html** — An interactive heatmap that shows all positions on the X axis and amino acids on the Y axis, and the predicted frustration for each mutation. Hovering over the heatmap shows raw values as a popup over the cursor.
* **frustrampnn.out.frustr.heatmap.svg** — A raw SVG file of the heatmap provided in the *frustrampnn.out.plotly.heamap.html* file.
