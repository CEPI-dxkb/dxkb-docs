# Influenza Reassortment Analysis Service

## Overview

The idea behind the influenza reassortment analysis tool [TreeSort](https://github.com/flu-crew/TreeSort) is the observation that if there is no reassortment, then the evolutionary histories of different segments should be identical. TreeSort uses a phylogenetic tree for one segment (e.g. the HA influenza A virus segment) as an evolutionary hypothesis for another segment (e.g. the NA segment). The first segment is referred to as the *reference* and the second as the *challenge*.

By trying to fit the sequence alignment of the challenge segment to the reference tree, TreeSort identifies points on that tree where this evolutionary hypothesis breaks. The "breaking" manifests in a mismatch between the divergence time on the reference tree (e.g. 1 year of divergence between sister clades) and an unlikely high number of substitutions in the challenge segment that would be required to explain the reference tree topology under the null hypothesis of no reassortment.

TreeSort has demonstrated very high accuracy in reassortment inference in simulations (manuscript in preparation). It can process datasets with tens of thousands of virus strains in just a few minutes, and can scale to very large datasets with hundreds of thousands of strains.

**Note:** The current version of TreeSort can ONLY be used with nucleotide sequences of an influenza virus. We hope to provide an updated version in the near future that can be used with common segmented viruses.

In the main menu, Influenza Reassortment Analysis is located under Tools & Services, in the Viral Tools category. You must be logged in to use this service.

## See also

* [Influenza Reassortment Analysis Service](https://dxkb.org/app/TreeSort)

## Input File

**NOTE: If the following instructions are not followed, your input file can't be submitted for analysis.**

TreeSort has very specific requirements for the format of strain names and segments in the headers of your FASTA file.

**Segment name**

* Influenza segment names in the FASTA file headers are limited to PB2, PB1, PA, HA, NP, NA, MP, and NS.
* The segment name must be within `|` characters with no whitespace around the name — for example, `|HA|`.

**Strain name / identifier.** The strain names in your FASTA headers can be formatted in one of three ways:

* The strain name is everything that remains after the segment name is removed.
* The strain name starts with `EPI_ISL_` followed by a numeric (integer) value.
* The strain name starts with A, B, C, or D followed by 3 to 5 spans of text inside `/` characters — for example, `A/swine/Iowa/A02635718/2021`. In this format, `|` characters are not allowed in the strain name.

**Additional requirements**

* Your FASTA file should have sequences for at least 10 strains (one sequence per segment).
* Sequences for at least 2 segments must be provided for each strain.
* Every strain should have a sequence for all segments referenced in the file. For example, if one strain has sequences for HA, NA, and PB1, all other strains in the file should have sequences for HA, NA, and PB1 as well.

As soon as you select a FASTA file from your workspace it is automatically validated. If there are problems with the file, an error message is displayed with suggestions about how to resolve them. If your FASTA file validates successfully, a summary of the number of strains and segments is displayed — please confirm that the summary is consistent with the contents of your file.

## Parameters

The parameters below configure the analysis. Reference Segment and Segments determine what is compared against what; Output Folder and Output Name determine where the results are written. The remaining parameters are grouped under Advanced options and can be left at their defaults for most analyses.

## Output Folder

The directory in your workspace where a directory will be created for the TreeSort results.

## Output Name

The name of the directory that will be created under the output folder. This name is also used for the primary results filename (*output name*.tre).

## Reference Segment

Reassortment events are acquisitions of one or more novel segments relative to this (fixed) reference segment.

## Segments

Select at least 2 segments to include in the analysis.

## Inference Method

The method used to place inferred reassortment events on the tree:

* **local** *(default)*
* **mincut** — The mincut method always determines the most parsimonious reassortment placement, even in ambiguous circumstances. It uses the reassortment test to cut the reference phylogeny into the optimum (smallest) number of non-reassorting parts, with theoretical guarantees on optimality. It is more robust than the local method in many instances, and does not produce "uncertain" reassortment inferences carrying the `?` annotation.

## Reference Tree Inference Method

The tool that will be used to infer the reference tree:

* **FastTree** *(default)* — Infers approximately-maximum-likelihood phylogenetic trees from alignments of nucleotide or protein sequences. Can handle alignments with up to a million sequences in a reasonable amount of time and memory. See [morgannprice.github.io/fasttree](https://morgannprice.github.io/fasttree/).
* **IQ-Tree** *(recommended for better accuracy)* — A fast search algorithm (Nguyen et al., 2015) to infer phylogenetic trees by maximum likelihood. See [iqtree.github.io](https://iqtree.github.io/).

## Allowed Deviation

Maximum deviation from the estimated substitution rate within each segment. The default is 2: the substitution rate on a particular tree branch is allowed to be twice as high or twice as low as the estimated rate. The default value was estimated from empirical influenza A data.

## P-value Threshold

The cutoff p-value for the reassortment tests; the default is 0.001 (0.1 percent). You may want to decrease or increase this parameter depending on how stringent you want the analysis to be.

## Clades Filename

The path to an output file where clades with evidence of reassortment will be saved.

## Estimate molecular clock rates for different segments

Estimate molecular clock rates for different segments, assuming equal rates.

## Collapse near-zero length branches into multifurcations

Collapse near-zero length branches into multifurcations. By default, TreeSort collapses all branches shorter than 1e-7 and then optimizes the multifurcations.

## Buttons

* **Reset:** Resets the input form to default values.
* **Submit:** Launches the analysis job. A message appears below the box to indicate that the job is now in the queue.

## Output Results

Clicking the Jobs indicator at the bottom of the page opens the Jobs Status page, which displays all current and previous service jobs and their statuses. Once the job has completed, you can view the results by double-clicking the job or clicking the View button on the green vertical Action Bar on the right-hand side of the page.

The Job Result page contains two sections:

* Details about the job including its ID, its start, end, and run times, and the parameters that were used when the job was submitted (click the arrow next to Parameters to view the job description formatted as JSON).
* Result files generated by the TreeSort service and saved in your workspace. The best place to start is the *index.html* page.

**Result files**

* **index.html** — An overview of the analysis results with links to all files generated by TreeSort, descriptions of the file types, and guidance on how to interpret the result data. It can be accessed by clicking the view icon (an eye) to the right of the Job Details, or by clicking the page icon to the left of *index.html* in the list of result files.
* **reassortments.csv** — A spreadsheet of the analysis results for every strain included in the input FASTA file. For each strain, the *is_reassorted* column contains a Y if reassortment is inferred, and the *is_uncertain* column indicates the uncertainty of the inference. The remaining columns represent the segments found in the input FASTA file; if reassortment is inferred for that strain, the reassorted segment's column contains the number of nucleotides that differ from the previous strain.
* **`<output name>`.xml** — An annotated tree file in PhyloXML format, where *output name* is the text entered in the Output Name field. Clicking this file's link in *index.html* opens it in the interactive Archaeopteryx viewer, providing advanced tree visualization tools.
* **`<output name>`.tre** — An annotated tree file in Nexus format.
* **files** (folder) — Segment-specific intermediate files generated by TreeSort before performing the analysis.

**Segment-specific files.** These are generated for every virus segment included in the analysis, where *segment name* is PB2, PB1, PA, HA, NP, NA, MP, or NS:

* `<segment name>`-input.fasta.aln
* `<segment name>`-input.fasta.aln.dates.csv
* `<segment name>`-input.fasta.aln.rooted.tre
* `<segment name>`-input.fasta.tre
* `<segment name>`-input.fasta.aln.treetime — a folder containing *outliers.tsv*, *root_to_tip_regression.pdf*, and *rtt.csv*

## References

* Markin A, Macken CA, Baker AL, Anderson TK. Revealing reassortment in influenza A viruses with TreeSort. *bioRxiv* 2024.11.15.623781. [doi:10.1101/2024.11.15.623781](https://doi.org/10.1101/2024.11.15.623781)
* TreeSort source: [github.com/flu-crew/TreeSort](https://github.com/flu-crew/TreeSort)
