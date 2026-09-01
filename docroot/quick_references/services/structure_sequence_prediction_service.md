# Structure Sequence Prediction Service (ProteinMPNN)

## Overview

This service runs [ProteinMPNN](https://github.com/dauparas/ProteinMPNN) on the dxkb servers. ProteinMPNN is a tool that takes a PDB file as input and predicts sequences that share a backbone with the given PDB file.

## See also

* [ProteinMPNN Service](https://dxkb.org/app/StructureSequencePrediction)

## PDB Selection

The protein selection option allows you to select a protein via:

* **Precomputed Structures:** When provided a [Protein Data Bank](https://www.rcsb.org) ID, the service will download the PDB directly from the Protein Data Bank and use that PDB in ProteinMPNN.
* **Upload a PDB File:** This option allows the use of a PDB file that has been uploaded to the user workspace. In the event the PDB file cannot be seen while selecting a PDB file, please ensure that the file is specified as a *pdb* type (or use the *show all files and folders* option while selecting).

## Parameters

There are various options which can be adjusted which affect the output of ProteinMPNN:

* **Model Name:** There are various models that are available for selection that follow the naming convention *v\_number graph edges\_backbone Ångström noise*. So v_48_002 would be a model graph with 48 edges trained with noise of 0.02. Depending on other options (like *CA Only*), not all models will be available with all options.
* **CA Only:** This option parses the CA-only structures and uses CA-only models.
* **Use Soluble Model:** This uses model weights that were trained on soluble proteins only.
* **Chains:** Specify which chains are to be designed. Leave it blank to design on all chains. For example, enter "A" to just do chain *A*, while "A,B,C" will run on chains *A*, *B*, and *C*.
* **Omit Amino Acids:** Specify which amino acids to omit in the sequence generation. A value of *X* will omit no amino acids, while *AC* would omit all *A* and *C*.
* **Backbone Noise:** This option adds the specified backbone noise to the predictions based on the standard deviation of Gaussian noise.
* **Number of Sequences per Target:** Select the number of sequences to generate for a given PDB file.
* **Sampling Temperature:** Specify the sampling temperature to use. Suggested values are 0.1, 0.15, 0.2, 0.25, and 0.3. Higher values will lead to more diversity.
* **PSSM Multi:** A value between [0.0, 1.0]. 0.0 means do not use PSSM; 1.0 means ignore MPNN predictions.

## Output

This selection specifies where the output of the service will be saved to the workspace. A workspace folder must be selected, and the output will be saved with the output name. The full path of the output would be *output folder* + *output name*.

## Service Output

The output of the ProteinMPNN service follows that of the ProteinMPNN tool. There is a folder named "out" and 3 folders within that:

* **probs** — Contains the probability scores for each of the amino acids for each of the positions within the sequence. This is saved as a NumPy array.
* **scores** — Contains the per-residue negative log-probabilities. This is saved as a NumPy array.
* **seqs** — Contains two files:
  * *protein name*.fa — Contains all of the sequence predictions made by ProteinMPNN. The first sequence is the original PDB sequence along with the parameters used in ProteinMPNN. The header of each prediction contains the temperature, sample (index), score, global score, and sequence recovery values.
  * *protein name*.fasta — Alignment of *protein name*.fa. This can also be viewed using the website's MSA viewer.
