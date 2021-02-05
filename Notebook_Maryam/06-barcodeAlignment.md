# Alignment of barcodes to the assembly

* Nova /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/06-barcodeAlignemnt
* Feb4, 2021

The purpose of this section is to confirm that there is no adaptor in each of the assemblies. The reason is we had to turn off trimming adaptors in the middle of reads in downpore ( Please check 01-QC section).

```bash
ln -s  ../01-QC/adapters_front.fasta .
ln -s  ../01-QC/adapters_back.fasta .

cat *.fasta > barcods-all.fasta
```

* run_alignment.bash
```
#!/bin/bash

barcode="$1"

module load minimap2


minimap2 -aLx map-ont barcodes-all.fasta   ../02-fly/out_flye_b$barcode/assembly.fasta  > aln-barcodes_$barcode\.sam

more aln-barcodes_$barcode\.sam | grep -v "@SQ" | awk '{print $1,$2,$3,$4,$6}'
```

Running the script through a for loop.

I did not find any alignment of barcodes on the assemblies.
