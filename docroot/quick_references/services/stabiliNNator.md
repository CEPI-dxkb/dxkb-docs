# Protein Stability Prediction Service

## Overview
The Protein Stability Prediction Service predicts protein stability improvements using Graph Neural Networks. The **proliNNator** model predicts proline mutation probabilities at each residue position, while the **disulfiNNate** model predicts disulfide bond formation likelihood between cysteine pairs. Both analyses output PDB files with the predicted probabilities encoded in the B-factor column (0–1 range), so results can be visualized directly in standard structure viewers.

## Parameters
Provide a protein structure and select the analysis (proline mutation prediction and/or disulfide bond prediction), then specify an output folder and output name. See the service input form for the full set of parameters.
