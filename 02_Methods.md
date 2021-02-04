# Methods


## Sample Collection

## Sequence Analysis
In order to estimate the insert locations, all strains have beed subject to nanopore sequenced (details about the instrument).   

## QC and trimming

The nextflow pipeline, NanoQCtrim (link to the github ?), have been used for quality control of the reads using Nanoplot ( version?) and then triming adaptors using downpore ( version?).

(Do I need a table here? stats for the reads?)

Because in few cases the barcodes were placed in the middle of the reads, the trimmer was trying to spit them twice in the middle which cause some errors. Therefore  we  used higher identity matching threshold of 100% (default is 85%) to detect adaptors. This way only adaptors that have a perfect match are trimmed. We might have a few adaptors that are not trimmed this way but we still think that it will not effect the quality of assembly. We will show that the quality of the assembly is still very good.






## Genome Assembly


## Alignment


## Dotplot
