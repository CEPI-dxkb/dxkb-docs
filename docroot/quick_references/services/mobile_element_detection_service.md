# Mobile Element Detection Service

## Overview

The Mobile Element Detection service allows you to identify viruses and plasmids in nucleic acid datasets. The use cases for this pipeline are broad — from detecting novel viruses in complex wastewater samples, to identifying plasmids in isolate genomes. The core of the Mobile Element Detection service is [geNomad](https://portal.nersc.gov/genomad/index.html), which detects viruses and plasmids in assembled contigs.

This pipeline can accept short reads or assembled contigs. If raw reads are provided, assembly is carried out first using any of the assembly methods in the Genome Assembly Service. The output of this pipeline is an assembled contigs file (if short reads are provided) and a tailored geNomad output which links identified viral and plasmid resources to dxkb databases.

## See also

* [Mobile Element Detection Service](https://dxkb.org/app/MobileElementDetection)
* [Genome Assembly Service](https://dxkb.org/app/Assembly2) — the assembly step used when reads are supplied

## Input Config

The service can accept either contigs (FASTA) or reads as input.

* **Contigs** — Select an assembled contigs file from the dropdown menu, or navigate to it in your workspace using the folder icon. geNomad runs directly on this file; no assembly step is performed.
* **Reads** — The form expands to add the read library options and the assembly parameters below. Paired-end libraries, single-end libraries, and SRA run accessions may all be supplied; the reads are assembled first, and geNomad then runs on the resulting contigs.

## Assembly Parameters

Shown only when *reads* is selected as the input type. These control the assembly that produces the contigs geNomad analyzes; they are the same parameters used by the Genome Assembly Service.

**Assembly Strategy** selects the assembler:

* *Auto* — Chooses the strategy based on the read libraries supplied. Recommended unless you have a reason to pick one.
* *Unicycler* — Hybrid and short-read isolate assembly.
* *SPAdes* — Short-read isolate assembly.
* *Canu* — Long-read (PacBio or Nanopore) assembly.
* *metaSPAdes* — Metagenomic short-read assembly.
* *plasmidSPAdes* — Targets plasmid sequences specifically.
* *MDA (single-cell)* — Single-cell / multiple displacement amplification data.
* *Flye* / *metaFlye* — Long-read isolate and metagenome assembly.
* *MegaHit* — Fast metagenomic short-read assembly.

## Read Processing

* **Normalize reads using BBNorm** — Apply BBNorm to reduce depth variation along the genome, down-sampling peaks of dense coverage to approximately the target coverage parameter.
* **Trim reads before assembly** — Trim reads using TrimGalore to remove any recognized Illumina adapters and low-quality 5' ends.
* **Filter long reads on length and quality** — Use FiltLong to down-sample Nanopore or PacBio reads to the target size (target genome coverage × estimated genome size).

## Genome Parameters

* **Estimated Genome Size** — Your estimate of how large the genome will be once assembled, in megabases (M) or kilobases (K). It is used to down-sample (BBNorm or FiltLong) to the desired target genome coverage, and is passed to Flye or Canu if either is selected as the assembler. It is ignored for metagenome methods.
* **Target Genome Coverage** — The target depth of reads mapped to the genome. 200X coverage is usually very good for assembling individual genomes. It is ignored for metagenome analyses.

## Assembly Polishing

**Racon Iterations** and **Pilon Iterations** correct assembly errors ("polish") using racon and/or Pilon. Both take the contigs and the reads mapped to those contigs and look for discrepancies between the assembly and the majority of the reads; where the majority of reads disagree with the assembly, the assembly is corrected. Racon is for long reads (PacBio or Nanopore) and Pilon is for Illumina reads. Once the assembly has been polished it is still possible to run another iteration to improve it further, but with less improvement each round — two rounds tend to approach saturation.

## Assembly Thresholds

* **Min. contig length** — Filter out contigs shorter than this value from the final assembly.
* **Min. contig coverage** — Filter out contigs with read depth lower than this value in the final assembly.
* **Maximum bases** — Cap on the total number of bases taken from the read libraries. Reads beyond this limit are not used, which bounds the run time of very large input sets.

## Genomad Parameters

These are passed through to geNomad and apply whether the input was contigs or assembled reads.

* **Filtering preset** — Adjusts how aggressively geNomad calls a contig viral or plasmid. *None* keeps geNomad's default thresholds; *Conservative* applies stricter cutoffs, producing fewer false positives at the cost of sensitivity; *Relaxed* loosens the cutoffs, finding more candidates with more false positives.
* **Sample composition** — Tells geNomad what kind of sample it is scoring, which sets the model priors. *Auto* estimates the composition from the data; *Metagenome* assumes a mixed community; *Virome* assumes a virus-enriched sample.
* **Force auto composition** — Forces the automatic composition estimate even when geNomad would otherwise decline to run it (for example on inputs it considers too small to estimate from reliably).
* **Lenient taxonomy** — Allows taxonomic assignment with weaker evidence, so more contigs receive a lineage but with lower confidence.
* **Full ICTV lineage** — Reports the complete ICTV lineage for assigned viruses rather than an abbreviated one.
* **Restart analysis** — Starts the geNomad run from the beginning rather than resuming from intermediate files.
* **Cleanup intermediate files** — Removes geNomad's intermediate working files from the output, keeping only the final results.
* **Verbose output** — Writes more detail to the job log; useful when diagnosing a run.
* **Debug level** — Additional diagnostic output, 0 (off) through 3.

## Output

Select an output folder for the results of the service and provide a name for the job output results. The full path of the output is *output folder* + *output name*.

## Selected Libraries

Read files placed here will contribute to a single analysis. Place read files here using the arrow buttons after selecting them above.

## Output Results

The service produces several files and folders:

* **Analysis_Summary.html** — Report summarizing the results of the service job. It has two sections: contigs identified as viral, then contigs identified as plasmids.
  * Contigs identified as viral are listed with details taken from geNomad — contig length, GC%, status (virus, provirus), geNomad assignment probability score, and taxonomy — alongside additional dxkb analysis. Phanotate annotations are carried out on identified viral sequences, linking these contigs to dxkb databases, which can be accessed through the Genome ID column for each viral contig.
  * Contigs identified as plasmids are listed with details taken from geNomad — contig length, GC%, topology, and geNomad assignment probability score — alongside additional dxkb analysis. RAST annotations are carried out on identified plasmid sequences, linking these contigs to dxkb databases, which can be accessed through the Genome ID column for each plasmid contig.
* **[report name]-contigs_virus_summary.tsv** — Tab-separated-value file containing a sample-wide summary of all viral contigs identified.
* **[report name]-contigs_plasmid_summary.tsv** — Tab-separated-value file containing a sample-wide summary of all plasmid contigs identified.
* **Viral Annotation** (folder) — For each virus contig, a folder containing a single FASTA file, allowing direct download or manipulation for further analysis.
* **Plasmid Annotation** (folder) — For each plasmid contig, a folder containing a single FASTA file, allowing direct download or manipulation for further analysis.

## References

* Camargo AP et al. Identification of mobile genetic elements with geNomad. *Nature Biotechnology* (2024). [doi:10.1038/s41587-023-01953-y](https://doi.org/10.1038/s41587-023-01953-y)
* geNomad documentation: [portal.nersc.gov/genomad](https://portal.nersc.gov/genomad/index.html)
