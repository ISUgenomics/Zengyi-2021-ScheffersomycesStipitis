## Alignments for barcode 16

####  Barcode16 (53) alignments:

* Nova : /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment

* Jan 26, 2021

I am aligning all the inserts on the assembly of strain `53` to find the location of inserts.

I look at the primary `flag 0` and reverse `flag 16` alignments in the sam files.


```bash
grep -v "@SQ" aln-b16.sam | awk '$3!="*"' | awk '$2==0' | awk '{print $1, $2, $3, $4, $6}'
```
```grep -v "@SQ" aln-b16.sam | awk '$3!="*"' | awk '$2==0' | awk '{print $1, $2, $3, $4, $6}'
6.BNU-loxP 0 contig_5 1596029 2078S95M1I2681M484S
1.BSA4upstream 0 contig_7 1979461 1673M1I11310M1I2080M
5.BTDN 0 contig_5 1590552 1065S182M2D2M2I522M4D7M2D17M1D11M3D2M1D7M1I16M1D10M4I2M1I2M4I3M2D3M1I1M1I4M1I7M7I3M1D12M2D7M1D5M1D16M2I7M1D5M2D4M1I1M3D4M1D4M1D7M4D11M2D2M1D6M3I9M4D2M5D3M1I8M1D6M1D10M2I1M3I8M4I4M5I18M4I2124M1I476M1I1964M1I1165M477S
4.BTDNU-loxP 0 contig_5 1590552 1065S182M2D2M2I522M4D7M2D17M1D11M3D2M1D7M1I16M1D10M4I2M1I2M4I3M2D3M1I1M1I4M1I7M7I3M1D12M2D7M1D5M1D16M2I7M1D5M2D4M1I1M3D4M1D4M1D7M4D11M2D2M1D6M3I9M4D2M5D3M1I8M1D6M1D10M2I1M3I8M4I4M5I18M4I2124M1I476M1I1964M1I2681M484S
3.GSA4 0 contig_7 1980025 1106S182M2I2M2D522M4I1M1I11M2I2M1D9M1I11M3I2M1I7M1D16M1I10M4D2M1D2M4D3M2I3M1D1M1D4M1D8M4D3M2D12M2I7M1I5M1I16M2D7M1I5M2I4M1D1M3I4M1I4M1I7M4I11M2I11M3I11M5I3M1D8M1I6M1I10M2D1M3D8M4D4M5D18M4D101M1I11310M1I2559M
2.GTDNUdownstream 0 contig_5 1589947 407M1I73M1I69M2D991M1I2195M1I476M1I1964M1I2681M11S
```

No alignment with flag of 16 (reverse alignment) was found.

For this strain we know that inserts `1.BSA4upstream` and `2.GTDNUdownstream` have been integrated.

for `1.BSA4upstream` the alignment shows all basepairs but 2 match the reference (CIGAR 1673M1I11310M1I2080M indicating two insertions and math for all other basepairs).

for `2.GTDNUdownstream` the alignment shows 6 insertions, 2 deletions and 11 soft clipping (CIGAR 407M1I73M1I69M2D991M1I2195M1I476M1I1964M1I2681M11S). Other alignments for this insert showed many more hard clipping. So I choose this as my correct alignment.

I now proceed to run alignments on read sections of the assembly near the insertion area.

I will use two different aligner to compare their results, minimap2 and gmap:

#### Minimap2

For `minimap2` aligner I use 1000 base pair down stream of the insertion location on the assembled genome:

* 1.BSA4upstream insert

from the sam file I know the insert location : contig_7 1979461

```bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b16

cp ../../../02-fly/out_flye_b16/assembly.fasta ./assembly-b16.fasta

module load samtools
samtools faidx assembly-b16.fasta

samtools faidx assembly-b16.fasta contig_7:1978461-1979461: > 1.BSA4upstream-c7-1978461-1979461-minimap.fasta
```

There is not 1000 basepairs in this contig after the insert. So I tried
```
samtools faidx assembly-b16.fasta contig_7:1978461-1995026 > 1.BSA4upstream-c7-1978461-1995026-gmap.fasta
```
* 2.GTDNUdownstream

From the sam file I know the location of insert: contig-5:1589947

```bash
samtools faidx assembly-b16.fasta contig_5:1588947-1589947 > 2.GTDNUdownstream-c5-1588947-1589947-minimap.fasta
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

from the sam file I know the insert location : contig_7 1979461

1979461+15065+1000=1995526

```bash
samtools faidx assembly-b16.fasta contig_7:1978461-1995526 > 1.BSA4upstream-c7-1978461-1995526-gmap.fasta
```

* 2.GTDNUdownstream

From the sam file I know the location of insert: contig-5:1589947

1589947+8873+1000=1599820

```bash
samtools faidx assembly-b16.fasta contig_5:1588947-1599820 > 2.GTDNUdownstream-c5-1588947-1599820-gmap.fasta
```

Concatenating the read sections:

```
cat *-gmap.fasta > inserts-gmap.fasta
cat *-minimap.fasta > inserts-minimap.fasta
```

Adding more details to the inset-IDs:

```
less inserts-gmap.fasta     | sed 's/contig_5/contig_5-6.2.GTDNUdownstream/g' | sed 's/contig_7/contig_7-1.BSA4upstream/g'  > inserts-gmap_correctID.fasta

less inserts-minimap.fasta     | sed 's/contig_5/contig_5-6.2.GTDNUdownstream/g' | sed 's/contig_7/contig_7-1.BSA4upstream/g'  > inserts-minimap_correctID.fasta
```

Running gmap and minimap2:


*  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/sub/minimap2_b16.sub

* /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/sub/gmap_b16.sub

alignment results from the sam files:

* /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b16/aln-b16-minimap.sam
* /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b16/b16-gmap.sam

```
grep -v "@SQ" aln-b16-minimap.sam  | awk '$3!="*"'  | awk '{print $1, $2, $3, $4, $6}'

```

| insert | gmap location | gmap CIGAR|  minimap location | minimap CIGAR|
| --- | --- | --- | ---| ---|
| 2.GTDNUdownstream| NA |NA|PQNB01000019.1 593474 | 2S999M |
| 1.BSA4upstream|NA| NA |NA|NA|


----------
The alignment of reference with assmebly made us believe that I need longer portion of assembly to align on the ref genome. Before I was using 100 bp upstream of the insertion position. I am going to try with 10k bp.

* 1.BSA4upstream insert

from the sam file I know the insert location : contig_7 1979461

```bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b16
```

```
samtools faidx assembly-b16.fasta contig_7:1978461-1995026 > 1.BSA4upstream-c7-1978461-1995026-gmap.fasta
```

--------
--------
#### Alignment of larger assembly sections on the reference genome using minimap

* Feb 1 , 2021
* Nova:/work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b14

I know the location of inserts. I select 10k bp upstream of the insertion from the assembly and align it on the reference.

```
samtools faidx assembly-b16.fasta contig_7:1969461-1979461  > 1.BSA4upstream-c7-1969461-1979461-minimap.fasta
samtools faidx assembly-b16.fasta contig_5:1579947-1589947  > 2.GTDNUdownstream-c5-1579947-1589947-minimap.fasta

 cat *-minimap.fasta > inserts-minimap.fasta

minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/GCA_006942115.1_ASM694211v1_genomic.fna  inserts-minimap.fasta > aln-b16.sam

grep -v "@SQ" aln-b16.sam | awk '$3!="*"'  | awk '{print $1, $2, $3, $4, $6}'

```
```
1.BSA4upstream-contig_7:1969461-1979461 16 PQNB01000001.1 778195 500S1728M1D6754M1D1019M
2.GTDNUdownstream-contig_5:1579947-1589947 16 PQNB01000019.1 593474 2S3226M1D539M1D6234M
```
