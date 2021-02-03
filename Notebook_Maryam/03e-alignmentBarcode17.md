## Alignments for barcode 17

####  Barcode17 (55) alignments:

* Nova : /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment

* Jan 26, 2021

I am aligning all the inserts on the assembly of strain `53` to find the location of inserts.

I look at the primary `flag 0` and reverse `flag 16` alignments in the sam files.


```bash
grep -v "@SQ" aln-b17.sam | awk '$3!="*"' | awk '$2==0' | awk '{print $1, $2, $3, $4, $6}'
```
```
5.BTDN 0 contig_27 1133064 1065S182M2D2M2I98M1I423M4D7M2D17M1D11M3D2M1D7M1I16M1D10M4I2M1I2M4I3M2D3M1I1M1I4M1I7M7I3M1D12M2D7M1D5M1D16M2I7M1D5M2D4M1I1M3D4M1D4M1D7M4D11M2D2M1D6M3I9M4D2M5D3M1I8M1D6M1D10M1D4M7I3M4I4M5I18M4I2367M1I233M1I1964M1I1165M477S
4.BTDNU-loxP 0 contig_27 1133064 1065S182M2D2M2I98M1I423M4D7M2D17M1D11M3D2M1D7M1I16M1D10M4I2M1I2M4I3M2D3M1I1M1I4M1I7M7I3M1D12M2D7M1D5M1D16M2I7M1D5M2D4M1I1M3D4M1D4M1D7M4D11M2D2M1D6M3I9M4D2M5D3M1I8M1D6M1D10M1D4M7I3M4I4M5I18M4I2367M1I233M1I1964M1I2683M482S
```
```
grep -v "@SQ" aln-b17.sam | awk '$3!="*"' | awk '$2==16' | awk '{print $1, $2, $3, $4, $6}'
```
```
6.BNU-loxP 16 contig_27 1123601 484S2777M2078S
1.BSA4upstream 16 contig_25 784604 2073M1I5827M1I2848M1I2635M1I1678M
3.GSA4 16 contig_25 784125 2552M1I5827M1I2848M1I2635M1I106M4D18M5D3M4D8M3D2M2D8M3I4M1D10M3I1M1I14M3I11M2I11M4I5M1I7M1I4M3I1M1D4M2I2M1I10M2D15M1I4M1I8M2I8M3D6M3D9M1D5M1D1M1D3M2I3M4D2M1D2M4D4M1I22M1D6M2I1M2I11M1I11M1D2M2I6M6I2M1D524M2D3M2I182M1106S
2.GTDNUdownstream 16 contig_27 1123601 11S4641M1I231M1I243M1I1794M1I402M1I993M1I3M1D62M1I66M1I213M1I205M
```

For this strain we know that inserts `1.BSA4upstream` and `2.GTDNUdownstream` have been integrated.

for `1.BSA4upstream` the alignment shows 4 insertions and the rest of base pair match (CIGAR 2073M1I5827M1I2848M1I2635M1I1678M).

for `2.GTDNUdownstream` the alignment shows 9 separate insertions , 11 soft clipping at the beginning and 1 deletion ( CIGAR 11S4641M1I231M1I243M1I1794M1I402M1I993M1I3M1D62M1I66M1I213M1I205M)

Other inserts don't show good alignment to the assembled as expected.  

I now proceed to run alignments on read sections of the assembly near the insertion area.

I will use two different aligner to compare their results, minimap2 and gmap:

#### Minimap2

For `minimap2` aligner I use 1000 base pair down stream of the insertion location on the assembled genome:

* 1.BSA4upstream insert

from the sam file I know the insert location : contig_25 784604

```bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b17

cp ../../../02-fly/out_flye_b17/assembly.fasta ./assembly-b17.fasta

module load samtools
samtools faidx assembly-b17.fasta

samtools faidx assembly-b17.fasta contig_25:783604-784604 > 1.BSA4upstream-c25-783604-784604-minimap.fasta

```
* 2.GTDNUdownstream

From the sam file I know the location of insert: contig-27:1123601

```bash
samtools faidx assembly-b17.fasta contig_27:1122601-1123601 > 2.GTDNUdownstream-c27-1122601-1123601-minimap.fasta
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

from the sam file I know the insert location : contig_25 784604

784604+15065+1000=800669

```bash
samtools faidx assembly-b17.fasta contig_25:783604-800669 > 1.BSA4upstream-c25-783604-800669-gmap.fasta
```
* 2.GTDNUdownstream

From the sam file I know the location of insert: contig-27:1123601

1123601+ 8873 + 1000 = 1133474

```bash
samtools faidx assembly-b17.fasta contig_27:1122601-1133474 > 1.BSA4upstream-c27-1122601-1133474-gmap.fasta
```
Concatenating the read sections:

```
cat *-gmap.fasta > inserts-gmap.fasta
cat *-minimap.fasta > inserts-minimap.fasta
```

Adding more details to the inset-IDs:

```
less inserts-gmap.fasta     | sed 's/contig_27/contig_27-2.GTDNUdownstream/g' | sed 's/contig_25/contig_25-1.BSA4upstream/g'  > inserts-gmap_correctID.fasta

less inserts-minimap.fasta     | sed 's/contig_27/contig_27.2.GTDNUdownstream/g' | sed 's/contig_25/contig_25-1.BSA4upstream/g'  > inserts-minimap_correctID.fasta
```

Running gmap and minimap2:


*  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/sub/minimap2_b17.sub

* /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/sub/gmap_b17.sub

alignment results from the sam files:

```bash
 grep -v "@SQ" aln-b17-minimap.sam  | awk '$3!="*"'  | awk '{print $1, $2, $3, $4, $6}'
 ```


| insert | gmap location | gmap CIGAR|  minimap location | minimap CIGAR|
| --- | --- | --- | ---| ---|
| 2.GTDNUdownstream| 199853 | !! |PQNB01000019.1 1060119  | 1S1000M|
| 1.BSA4upstream|NA| NA | PQNB01000001.1 777685|517M484S|

--------
--------
#### Alignment of larger assembly sections on the reference genome using minimap

* Feb 1 , 2021
* Nova:/work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b14

I know the location of inserts. I select 10k bp upstream of the insertion from the assembly and align it on the reference.

```
samtools faidx assembly-b17.fasta contig_25:774604-784604  > 1.BSA4upstream-c25-774604-784604-minimap.fasta
samtools faidx assembly-b17.fasta contig_27:1113601-1123601  > 2.GTDNUdownstream-c27-1113601-1123601-minimap.fasta
samtools faidx assembly-b17.fasta contig_27:1122463-1132463  > 2.GTDNUdownstream-c27_2-1122463-1132463-minimap.fasta

cat *-minimap.fasta > inserts-minimap.fasta

minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/GCA_006942115.1_ASM694211v1_genomic.fna  inserts-minimap.fasta > aln-b17.sam

grep -v "@SQ" aln-b17.sam | awk '$3!="*"'  | awk '{print $1, $2, $3, $4, $6}'
```

```
1.BSA4upstream-contig_25:774604-784604 0 PQNB01000001.1 768685 9517M484S
2.GTDNUdownstream-contig_27:1113601-1123601 16 PQNB01000019.1 1060119 1S7591M1D2269M1D140M
2.GTDNUdownstream_2-contig_27:1122463-1132463 16 PQNB01000006.1 621765 7346S146M3D1131M1D211M1167S
2.GTDNUdownstream_2-contig_27:1122463-1132463 2064 PQNB01000019.1 1060119 8863H1138M
2.GTDNUdownstream_2-contig_27:1122463-1132463 2048 PQNB01000001.1 333817 9396H38M1I11M1D62M1D66M1D213M1D205M9H
2.GTDNUdownstream_2-contig_27:1122463-1132463 2064 PQNB01000001.1 339391 1621H4M1D326M1D237M7813H
2.GTDNUdownstream_2-contig_27:1122463-1132463 2064 PQNB01000001.1 1961659 6087H547M3367H
2.GTDNUdownstream_2-contig_27:1122463-1132463 2064 PQNB01000006.1 526937 3984H233M1D301M5483H
2.GTDNUdownstream_2-contig_27:1122463-1132463 2064 PQNB01000006.1 528580 3674H68M1D236M6023H
2.GTDNUdownstream_2-contig_27:1122463-1132463 2048 PQNB01000022.1 719752 8383H66M1D233M1319H
2.GTDNUdownstream_2-contig_27:1122463-1132463 2048 PQNB01000006.1 626849 3920H150M5931H
2.GTDNUdownstream_2-contig_27:1122463-1132463 2048 PQNB01000023.1 405895 2698H155M7148H
```
