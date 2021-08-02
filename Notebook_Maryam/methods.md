# Methods


## Sample Collection

## Sequence Analysis
In order to estimate the insert locations, all strains have beed subject to nanopore sequencing using [GridION X5](https://nanoporetech.com/products/gridion) from Oxford Nanopore Technologies. For each strain we assembled a genome and locate the inserts on that genome as will be discussed further bellow.   

## QC and trimming

The [nextflow](https://www.nextflow.io/) workflow, [NanoQCtrim](https://github.com/isugifNF/nanoQCtrim), was used for quality control and trimming of the reads. This workflow uses [NanoPlot](https://github.com/wdecoster/NanoPlot) (v. 1.32.0) for quality control and trims adaptors using [downpore](https://github.com/jteutenberg/downpore) (v. 0.3.3). Default parameters were used for Nanoplot.  For Downpore, the identity matching threshold for the adaptors found in the middle  was increased to 100% (default is 85%).  No adapter sequences were identified in the final assemblies based on a minimp2 alignment of adapters using default parameters.

## Assembly

We used Flye (v.2.8.2-b1691) for de novo assembly of each of the strain's genomes. Quality of the assemblies were assessed using the nextflow pipeline [assemblyStats](https://github.com/isugifNF/assemblyStats). The pipeline provides some useful statistics about the assembly as well as [BUSCO](https://busco.ezlab.org/) score. Busco version of v4.1.4_cv1 with the Fungi library (fungi_odb10) was used to test for the completeness of the genome assembly.

## Alignment

[Minimap2](https://github.com/lh3/minimap2) (v. 2.2-r409) was used for all the alignments performed during this study. The known insert sequences were aligned to each assembled genome. Upstream sequence of each aligned insert to the assembled genome was extracted (10,000 bases) and then aligned to the reference genome (ASM694211v1) to identify the insert location in the reference genome.


## Comparing genomes using dotplot

We used [re-DOT-able](https://www.bioinformatics.babraham.ac.uk/projects/redotable/) to compare the reference genome and assembled genome for each strain.
=======
In order to visualize pairwise comparison between sequences we used a desktop app ([re-DOT-able](https://www.bioinformatics.babraham.ac.uk/projects/redotable/) ) from Babraham Bioinformatics to plot  interactively and also save the plots as images. We made pairwise comparisons between the reference genome and each of the seven assembled genomes to look for insert location and for any genome re-arrangements. In order to check the location Additionally we also plotted each assembly against the relevant insert sequences. For detailed information; Simon Andrews, the developer has a really good [video tutorial](https://www.youtube.com/watch?v=qPxl2hflG9Q&feature=emb_logo).

