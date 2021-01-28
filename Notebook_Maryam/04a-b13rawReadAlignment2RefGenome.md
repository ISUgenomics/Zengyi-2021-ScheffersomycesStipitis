# Alignment of the raw reads to the reference genome

* Jan 27, 2021

The goal is to confirm the position of the inserts on the assembled genomes. To to that I am going to align all the raw reads to the inserts first to filter the reads that partially contain the inserts.

In the next step I will align the filter reads on the reference genome and I expect to see a peak at the assumed insert position on the reference genome. The assumption is that the part of those reads that do not contain the insert , will align near the insertion location.
We filter the reads to reduce the noise.


Lets start:
* Nova /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/

```bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/04-RawReadAlignments/b13
cp ../../00-RawData/GTDNUdownstream.fasta .
mv GTDNUdownstream.fasta 2.GTDNUdownstream.fasta
cp ../../00-RawData/BSA4upstream.fasta .
mv BSA4upstream.fasta 1.BSA4upstream.fasta
```

I am going to start with `1.BSA4upstream`.

##### 1.BSA4upstream
```
ln -s ../../00-RawData/barcode13/combine-barcode13.fastq .

```

running alignment of raw reads to the insert.

```bash

```

Now I get the list of the reads that have aligned to the insert.

```bash
less aln-b13.sam | grep -v "@" |  awk '{print $1}' > name.list
```
I want to check how many of the raw reads were aligned to the insert:
```bash
less aln-b13.sam | grep -v "@" | wc
   406    6349  287995
```

Now extract the reads from the list from the raw reads.

```bash
module load seqtk
seqtk subseq combine-barcode13.fastq name.list > rawread-sublist.fastq
```

333817 + 15065 = 348882
333817 + 8873 = 342690
