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

## Assembly with Flye genome assembler

Runing flye

```bash
for n in 4 5 6 7 8 9 ; do
flye --nano-raw /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/01-QC/output-barcode$n/trimmedReads/FAO68114_pass_barcode$n_598c81f2_0_adaptersRemoved.fastq --out-dir out_barcode$n
 --genome-size 15m --threads 30 -i 4
 ```

## Allignmnet

### Alliging the insert onto the assemblies with minimap2

```bash
for n in 4 5 6 7 8 9 ; do
minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData/inserts-all.fasta  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/02-fly/out_
flye_b$n/assembly.fasta  > aln-b$n\.sam " >> minimap2_$n
```
