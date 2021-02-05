# Methods


## Sample Collection

## Sequence Analysis
In order to estimate the insert locations, all strains have beed subject to nanopore sequencing using [GridION X5](https://nanoporetech.com/products/gridion) from Oxford Nanopore Technologies. For each strain we assembles a genome and locate the insert on that genome as will be discussed further bellow.   

## QC and trimming

The nextflow pipeline, [NanoQCtrim] (https://github.com/isugifNF/nanoQCtrim), have been used for quality control and trimming of the reads. Pipeline uses Nanoplot (v. 1.32.0) for quality control and then trims the  adaptors using downpore (v. 0.3.3).

We had to increase identity matching threshold for the adaptors found in the middle to 100% (default is 85%). In few cases downpore failed to handle trimming those adaptors and crashed. This increase will prevent trimming of the adaptors that are in the middle of the reads but are not 100% match. We believe that leaving a few adopters in the middle if the reads will not effect the quality of assembly. To prove this we aligned each of the adaptors to the assembled genomes and confirmed that no adaptor is present in the assembly.

## Assembly

We used Flye (v.2.8.2-b1691) for de novo assembly of each of the strain's genomes. Quality of the assemblies were assessed using the nextflow pipeline [assemblyStats](https://github.com/isugifNF/assemblyStats). The pipeline provides some useful statistics about the assembly as well as Busco score. Busco version of v4.1.4_cv1 with the Fungi library (fungi_odb10) have been used to test for the completeness of the genome assembly.

## Alignment

Minimap2 (v. 2.2-r409) have been used for all the alignments performed during this study. For each strain we aligned two inserts on the assembled genome to estimate the insert location on the assembly. Next we used 10K base pair upstream fo the insert location from the assembly and aligned it to the reference genome ASM694211v1 to estimate the insert location on the reference genome.

## Comparing genomes using dotplot
We have used [re-DOT-able](https://www.bioinformatics.babraham.ac.uk/projects/redotable/) to compare the reference genome and assembled genome for each strain. 





## Genome Assembly


## Alignment


## Dotplot
