# Stability Prediction Service (ThermoMPNN)

## Overview

The dxkb ThermoMPNN-D service allows you to run [ThermoMPNN-D](https://github.com/Kuhlman-Lab/ThermoMPNN-D) on the dxkb servers. ThermoMPNN-D takes a PDB file and makes every possible singleton and/or dualton mutation possible, and computes the free energy of protein folding caused by the respective mutations, ddG (kcal/mol). The output is in the form of a CSV file.

## See also

* [ThermoMPNN Service](https://dxkb.org/app/StabilityPrediction)

## PDB Selection

The protein selection option allows you to select a protein via:

* **Precomputed Structures:** When provided a [Protein Data Bank](https://www.rcsb.org) ID, the service will download the PDB directly from the Protein Data Bank and use that PDB in ThermoMPNN-D.
* **Upload a PDB File:** This option allows the use of a PDB file that has been uploaded to the user workspace. In the event the PDB file cannot be seen while selecting a PDB file, please ensure that the file is specified as a *pdb* type (or use the *show all files and folders* option while selecting).

## Parameters

There are various options which can be adjusted which affect the output of ThermoMPNN-D:

* **SSM Mode:**
  * *Single* — Evaluates single point mutations.
  * *Additive* — Evaluates double point mutations by adding together two single point mutations. It serves as a fairly naive approximation but runs quickly.
  * *Epistatic* — More intense prediction which looks at the interaction between two point mutations. It is more accurate than the simple additive model, but takes longer to run.
* **SS Penalty:** Allows one to decide if there is a penalty when a mutation breaks a disulfide bond.
* **Chains:** Specify what chains to mutate over. Leave this blank to do all chains, or specify specific chains. For example, enter "A" to just do chain *A*, while "A B C" will run on chains *A*, *B*, and *C*.
* **Threshold:** Limit the output of ThermoMPNN based on a ddG threshold. 100 will save all values, while values lower than 100 will only allow mutations with ddG > the threshold to be saved.
* **Distance:** Used in double mutation ddG predictions; will predict on double mutations whose CA distance is < the given distance threshold.

## Output

This selection specifies where the output of the service will be saved to the workspace. A workspace folder must be selected, and the output will be saved with the output name. The full path of the output would be *output folder* + *output name*.

## Service Output

There is a single file output for the ThermoMPNN-D service; it is a *CSV* file produced by ThermoMPNN-D. It contains a header with 4 columns:

* *(no label)* — The unlabeled column is an index column that counts up sequentially from 0.
* *ddG (kcal/mol)* — Free energy prediction based on the mutation.
* *Mutation* — The mutation performed. For dual mutations, mutations are separated by a colon ":".
* *CA-CA Distance* — For dual mutations, a CA-CA distance is also provided.
