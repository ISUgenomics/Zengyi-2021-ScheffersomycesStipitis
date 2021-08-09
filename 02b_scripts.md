# Scripts used to run the analysis

## Quallity control

Running all barcodes in a loop

```bash
for n in 4 5 6 7 8 9 ; do   nextflow run /work/gif/Maryam/Programfiles/nanoQCtrim/main.nf --fastqs ../00-RawData/barcode1$n/combine-barcode1$n.fastq   --outdir output-barcode1$n  --options '-middle_threshold 100' -profile nova,singularity; done
```

where
* main.nf
```bash

```

## Assembly of each line with Flye genome assembler


Runing flye

```bash
for n in 4 5 6 7 8 9 ; do
flye --nano-raw /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/01-QC/output-barcode$n/trimmedReads/FAO68114_pass_barcode$n_598c81f2_0_adaptersRemoved.fastq --out-dir out_barcode$n
 --genome-size 15m --threads 30 -i 4
 ```

## Alignment

### Aligning the inserts on the assemblies with minimap2

```bash
for n in 4 5 6 7 8 9 ; do
minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData/inserts-all.fasta  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/02-fly/out_
flye_b$n/assembly.fasta  > aln-b$n\.sam " >> minimap2_$n
```

### Selecting 10K bp upstream from the insert location on each assembly

We used `samtools` to extract 10kb upstream sections of the assembly for each insert location. We aligned back these sections on the reference genome to estimate the insert locations on the reference genome.

### Aligning the 10kb upstream (from the insert location) sections of the assemblies on the reference genome with minimap2

```bash
for n in 4 5 6 7 8 9 ; do
minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/GCA_006942115.1_ASM694211v1_genomic.fna  inserts$n-minimap.fasta > aln$n.sam
```
From the sam files we can estimate the insert locations on the reference genome.
