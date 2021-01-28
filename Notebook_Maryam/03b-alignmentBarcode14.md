## Alignments for barcode 15

####  Barcode14 (NC22) alignments:

I look at the primary `flag 0` and reverse `flag 16` alignments in the sam files.


```bash
grep -v "@SQ" aln-b14.sam | awk '$3!="*"' | awk '$2==16' | awk '{print $1, $2, $3, $4, $6}'
```
`6.BNU-loxP 16 contig_4 123838 3160M1I100M2078S
1.BSA4upstream 16 contig_6 3421872 2073M1I8676M1I2742M4I18M5I3M4I3M7I5M2D8M3D4M1I10M3D1M1D14M3D11M2D11M4D5M1D7M1D4M3D1M1I4M2D2M1D10M2I15M1D4M1D8M2D8M3I6M3I9M1I5M1I1M1I3M2D3M4I2M1I2M4I4M1D22M1I6M2D1M2D11M1D19M2D7M4D521M2I3M2D182M564S
5.BTDN 16 contig_4 125838 477S1160M1I6234M2I408M
4.BTDNU-loxP 16 contig_4 123838 3160M1I6234M2I408M
3.GSA4 16 contig_6 3421393 2552M1I8676M1I4857M
2.GTDNUdownstream 16 contig_4 124311 2687M1I4571M4D18M5D3M4D8M3D2M2D8M3I4M1D10M3I1M1I14M3I11M2I11M4I5M1I7M1I4M3I1M1D4M2I2M1I10M2D15M1I4M1I8M2I8M3D6M3D9M1D5M1D1M1D3M2I3M4D2M1D2M4D4M1I22M1D6M2I1M2I11M1I11M1D2M2I6M6I2M1D524M2D3M2I182M605S
`
primary alignment search (flag=0) didn't result in any alignment.

The inserts in this strain are
`4.BTDNU-loxP` and `3.GSA4`.

For `3.GSA4` I checked and this is the only alignment.

```
grep -v "@SQ" aln-b14.sam | awk '$3!="*"' | grep "3.GSA4"  | awk '{print $1, $2, $3, $4, $6}'
3.GSA4 16 contig_6 3421393 2552M1I8676M1I4857M
```

This is a good alignment with only 2 insertions and all the other base pairs match.

for `4.BTDNU-loxP` also the alignment is good ( all the base pairs other than 3 match).

`4.BTDNU-loxP 16 contig_4 123838 3160M1I6234M2I408M`

Other alignments include soft or hard clipping with very lower rate of match between base pairs. This conforms that only two of the inserts have been randomly integrated as expected.

I now proceed to run alignments on read sections of the assembly near the insertion area.

First I use minimap to find alignemnt of a section of the assembly from  x-1000 bp untill x bp.

* 3.GSA4 (x=contig_16:3421393)


```#!/usr/bin/env bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b14

cp ../../../02-fly/out_flye_b14/assembly.fasta ./assembly-b14.fasta

module load samtools

samtools  faixd assembly-b14.fasta

samtools faidx assembly-b14.fasta contig_6:3420393-3421393 > GSA4-c6-3420393-3421393-minimap.fasta

```


Then `gmap` to align the section between x-1000 and x+insertLenght+1000

```
grep -v ">" GSA4_sequence.fasta | wc | awk '{print $3-$1}'
  16087
```
insert length = 16087
 3420393+ 16087 + 1000 = 3437480

 ```bash
 samtools faidx assembly-b13.fasta contig_6:3420393-3437480 > GSA4-c6-3420393-3437480-gmap.fasta
 ```

* 4.BTDNU-loxP (contig-4:123838)

```
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData
grep -v ">" BTDNU-loxP_sequence.fasta | wc | awk '{print $3-$1}'
9805
```
for gmap :
122838 +9805 + 1000 = 133643

```
/work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b14

samtools faidx assembly-b14.fasta contig_4:122838-123838 > BTDNU-loxP-c4-122838-123838-minimap.fasta

samtools faidx assembly-b14.fasta contig_4:122838-133643 > BTDNU-loxP-c4-122838-133643-gmap.fasta

```

Concatenating the read sections:
```
cat *-gmap.fasta > inserts-gmap.fasta
cat *-minimap.fasta > inserts-minimap.fasta
```
Adding more details to the inset-IDs:

```
less inserts-gmap.fasta     | sed 's/contig_4/contig_4-4.BTDNU-loxP/g' | sed 's/contig_6/contig_6-3.GSA4/g
'  > inserts-gmap-correctIDs.fasta

less inserts-minimap.fasta     | sed 's/contig_4/contig_4-4.BTDNU-loxP/g' | sed 's/contig_6/contig_6-3.GSA
4/g'  > inserts-minimap-correctIDs.fasta
```

Running gmap and minimap2 with script mentioned above.

```
grep -v "@SQ" aln-b14-minimap.sam  | awk '$3!="*"'  | awk '{print $1, $2, $3, $4}'
```

| insert | gmap location | gmap CIGAR|  minimap location | minimap CIGAR|
| --- | --- | --- | ---| ---|
| 4.BTDNU-loxP | 199853 |!!!| PQNB01000005.1 629073 |17M1D984M |
| 3.GSA4 | NA | NA |PQNB01000019.1 2089204|561M440S|
