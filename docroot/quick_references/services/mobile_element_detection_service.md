# Mobile Element Detection Service

## Overview
The Mobile Element Detection service allows you to identify viruses and plasmids in nucleic acid datasets. The use cases for this pipeline are broad – from detecting novel viruses in complex wastewater samples, to identifying plasmids in isolate genomes. The core of the Mobile Element Detection service is [geNomad](https://portal.nersc.gov/genomad/index.html) which detects viruses and plasmids in assembled contigs. This pipeline can accept short reads or assembled contigs. If raw reads are provided, assembly can be carried out using any of the assembly methods in the Genome Assembly Service. The output of this pipeline is an assembled contigs file (if short reads are provided) and a tailored geNomad output which links identified viral and plasmid resources to dxkb databases.

## Parameters
See the service input form for the full set of parameters. Provide either short reads (which will be assembled) or assembled contigs, then specify an output folder and output name.
