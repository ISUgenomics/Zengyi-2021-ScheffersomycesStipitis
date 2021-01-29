## Alignments for barcode 18

####  Barcode18 (55) alignments:

* Nova : /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment

* Jan 26, 2021
I am aligning all the inserts on the assembly of strain `59` to find the location of inserts.

I look at the primary `flag 0` and reverse `flag 16` alignments in the sam files.

```bash
grep -v "@SQ" aln-b18.sam | awk '$3!="*"' | awk '$2==0' | awk '{print $1, $2, $3, $4, $6}'
```
```
6.BNU-loxP 0 scaffold_10 902609 2078S95M1I2695M470S
5.BTDN 0 scaffold_10 897132 1065S182M2D2M2I522M4D7M2D17M1D11M3D2M1D7M1I16M1D10M4I2M1I2M4I3M2D3M1I1M1I4M1I7M7I3M1D12M2D7M1D5M1D16M2I7M1D5M2D4M1I1M3D4M1D4M1D7M4D11M2D11M3D11M5D3M1I8M1D6M1D10M2I1M3I8M4I4M5I18M4I2124M1I324M1I151M1I1964M1I1165M477S
4.BTDNU-loxP 0 scaffold_10 897132 1065S182M2D2M2I522M4D7M2D17M1D11M3D2M1D7M1I16M1D10M4I2M1I2M4I3M2D3M1I1M1I4M1I7M7I3M1D12M2D7M1D5M1D16M2I7M1D5M2D4M1I1M3D4M1D4M1D7M4D11M2D11M3D11M5D3M1I8M1D6M1D10M2I1M3I8M4I4M5I18M4I2124M1I324M1I151M1I1964M1I2695M470S
2.GTDNUdownstream 0 scaffold_10 896527 3738M1I324M1I151M1I1964M1I2692M
```

The only good primary alignment is for `2.GTDNUdownstream` with only 4 insertions (CIGAR 896527 3738M1I324M1I151M1I1964M1I2692M)


```bash
grep -v "@SQ" aln-b18.sam | awk '$3!="*"' | awk '$2==16' | awk '{print $1, $2, $3, $4, $6}'
```
```
1.BSA4upstream 16 contig_9 784693 13386M1I1678M
3.GSA4 16 contig_9 784214 13865M1I106M4D18M5D3M4D8M3D2M2D8M3I4M1D10M3I1M1I14M3I11M2I11M4I5M1I7M1I4M3I1M1D4M2I2M1I10M2D15M1I4M1I8M2I8M3D6M3D9M1D5M1D1M1D3M2I3M4D2M1D2M4D4M1I22M1D6M2I1M2I11M1I11M1D2M2I6M6I2M1D524M2D3M2I182M1106S
```
The only good reverse alignment is for `1.BSA4upstream` with only `1` inertions (CIGAR 13386M1I1678M).

As expected only `1.BSA4upstream` and `2.GTDNUdownstream` has good alignments to the assembled genomes. Only these two inserts were integrated.


I now proceed to run alignments on read sections of the assembly near the insertion area.

I will use two different aligner to compare their results, minimap2 and gmap:

#### Minimap2

For `minimap2` aligner I use 1000 base pair down stream of the insertion location on the assembled genome:

* 1.BSA4upstream insert

from the sam file I know the insert location : contig_9 784693

```bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b18

cp ../../../02-fly/out_flye_b18/assembly.fasta ./assembly-b18.fasta

module load samtools
samtools faidx assembly-b18.fasta

samtools faidx assembly-b18.fasta contig_9:783693-784693 > 1.BSA4upstream-c9-783693-784693-minimap.fasta

```

* 2.GTDNUdownstream

From the sam file I know the location of insert is scaffold_10 896527:

```bash
samtools faidx assembly-b18.fasta s10:895527-896527 > 2.GTDNUdownstream-s10-895527-896527-minimap.fasta
```

#### gmap

For `gmap` aligner I use 1000 base pair before the insertion location on the assembled genome plus insert length plus 1000 basepairs after the insert. First I need to know the insert length :

```bash
grep -v ">" BSA4upstream.fasta | wc | awk '{print $3-$1}'
  15065

grep -v ">" GTDNUdownstream.fasta | wc | awk '{print $3-$1}'
  8873
```

* 1.BSA4upstream insert

from the sam file I know the insert's location is contig_9 784693 :

784693+15065+1000=800758

```bash
samtools faidx assembly-b18.fasta contig_9:783693-800758> 1.BSA4upstream-c9-783693-800758-gmap.fasta
```
* 2.GTDNUdownstream

From the sam file I know the location of insert is scaffold_10 896527:

896527+ 8873 + 1000 = 906400

```bash
samtools faidx assembly-b18.fasta scaffold_10:895527-906400 > 1.BSA4upstream-s10-895527-906400-gmap.fasta
```
Concatenating the read sections:

```
cat *-gmap.fasta > inserts-gmap.fasta
cat *-minimap.fasta > inserts-minimap.fasta
```
Adding more details to the inset-IDs:

```less inserts-gmap.fasta     | sed 's/contig_9/contig_9-2.GTDNUdownstream/g' | sed 's/scaffold_10/scaffold_10-1.BSA4upstream/g'  > inserts-gmap_correctID.fasta

less inserts-minimap.fasta     | sed 's/contig_9/contig_9.2.GTDNUdownstream/g' | sed 's/scaffold_10/scaffold_10-1.BSA4upstream/g'  > inserts-minimap_correctID.fasta
```
Running gmap and minimap2:


*  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/sub/minimap2_b18.sub

* /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/sub/gmap_b18.sub

alignment results from the sam files:

* minimap error :`minimap2: sketch.c:83: mm_sketch: Assertion `len > 0 && (w > 0 && w < 256) && (k > 0 && k <= 28)' failed.
Aborted (core dumped)`

```bash
grep -v "@SQ" aln-b18-minimap.sam  | awk '$3!="*"'  | awk '{print $1, $2, $3, $4, $6}'
```

| insert | gmap location | gmap CIGAR|  minimap location | minimap CIGAR|
| --- | --- | --- | ---| ---|
| 2.GTDNUdownstream| NA |NA |NA|NA|
| 1.BSA4upstream|526936|!!!!  |NA|NA |
