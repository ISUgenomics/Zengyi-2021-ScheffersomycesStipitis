## Alignments for barcode 18

####  Barcode18 (55) alignments:

* Nova : /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment

* Jan 27, 2021

I am aligning all the inserts on the assembly of strain `88` to find the location of inserts.

I look at the primary `flag 0` and reverse `flag 16` alignments in the sam files.

```bash
grep -v "@SQ" aln-b19.sam | awk '$3!="*"' | awk '$2==0' | awk '{print $1, $2, $3, $4, $6}'

```
```
6.BNU-loxP 0 contig_7 3079527 2078S95M1I2691M474S
1.BSA4upstream 0 contig_6 1970041 1673M1I4674M1I317M1I8398M
5.BTDN 0 contig_7 3074049 1065S182M2D2M2I522M4D7M2D17M1D11M3D2M1D7M1I16M1D10M4I2M1I2M4I3M2D3M1I1M1I4M1I7M7I3M1D12M2D7M1D5M1D16M2I7M1D5M2D4M1I1M3D4M1D4M1D7M4D11M2D11M3D11M5D3M1I8M1D6M1D10M2I1M3I8M4I4M5I18M4I2124M1I476M1I1964M1I1165M477S
4.BTDNU-loxP 0 contig_7 3074049 1065S182M2D2M2I522M4D7M2D17M1D11M3D2M1D7M1I16M1D10M4I2M1I2M4I3M2D3M1I1M1I4M1I7M7I3M1D12M2D7M1D5M1D16M2I7M1D5M2D4M1I1M3D4M1D4M1D7M4D11M2D11M3D11M5D3M1I8M1D6M1D10M2I1M3I8M4I4M5I18M4I2124M1I476M1I1964M1I2691M474S
3.GSA4 0 contig_6 1970605 1106S182M2I2M2D522M4I1M1I11M2I2M1D9M1I11M3I2M1I7M1D16M1I10M4D2M1D2M4D3M2I3M1D1M1D4M1D8M4D3M2D12M2I7M1I5M1I16M2D7M1I5M2I4M1D1M3I4M1I4M1I7M4I11M2I11M3I11M5I3M1D8M1I6M1I10M2D1M3D8M4D4M5D18M4D101M1I4674M1I317M1I8877M
2.GTDNUdownstream 0 contig_7 3073445 407M1I3330M1I476M1I1964M1I2691M1S

```

From the primary alignments insert `1.BSA4upstream` has a good alignment with only 3 insertions  at position  `contig_6 1970041` (CIGAR :  1673M1I4674M1I317M1I8398M)

Also insert `2.GTDNUdownstream` has a good alignment at position `contig_7 3073445` (CIGAR :1673M1I4674M1I317M1I8398M)

No reverse alignment (flag 16) was reported in the sam files.

I now proceed to run alignments on read sections of the assembly near the insertion area.

I am using only minimap2 because we decided that `gmap` is not suitable for long noisy reads.

#### Minimap2

For `minimap2` aligner I use 1000 base pair up stream of the insertion location on the assembled genome:

* 1.BSA4upstream insert

from the sam file I know the insert location : contig_6 1970041

```bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b19

cp ../../../02-fly/out_flye_b19/assembly.fasta ./assembly-b19.fasta

module load samtools
samtools faidx assembly-b19.fasta

samtools faidx assembly-b19.fasta contig_6:1969041-1970041 > 1.BSA4upstream-c6-1969041-1970041-minimap.fasta

```

* 2.GTDNUdownstream

From the sam file I know the location of insert is :contig_7 3073445

```bash
samtools faidx assembly-b19.fasta contig_7:3072445-3073445 > 2.GTDNUdownstream-c7:3072445-3073445-minimap.fasta
```

Concatenating the read sections:

```
cat *-minimap.fasta > inserts-minimap.fasta
```

Adding more details to the inset-IDs:
```
less inserts-minimap.fasta     | sed 's/contig_7/contig_7.2.GTDNUdownstream/g' | sed 's/contig_6/contig_6-1.BSA4upstream/g'  > inserts-minimap_correctID.fasta
```

Running  minimap2:

*  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/sub/minimap2_b18.sub

```
minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/GCA_006942115.1_ASM694211v1_genomic.fna /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b19/inserts-minimap_correctID.fasta   > /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b19/aln-b19-minimap.sam
```

alignment results from the sam file:
```bash
grep -v "@SQ" aln-b19-minimap.sam  | awk '$3!="*"'  | awk '{print $1, $2, $3, $4, $6}'
```
```
contig_6-1.BSA4upstream:1969041-1970041 16 PQNB01000001.1 778195 499S39M1I462M
contig_7.2.GTDNUdownstream:3072445-3073445 16 PQNB01000028.1 440888 2S999M
```
| insert |   minimap location | minimap CIGAR|
| --- | ---| ---|
| 1.BSA4upstream|PQNB01000001.1 778195 |499S39M1I462M|
| 2.GTDNUdownstream| PQNB01000028.1 440888 |2S999M|

both are very good alignments based on their CIGAR.


--------
--------
#### Alignment of larger assembly sections on the reference genome using minimap

* Feb 2 , 2021
* Nova:/work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b14

I know the location of inserts. I select 10k bp upstream of the insertion from the assembly and align it on the reference.

```
samtools faidx assembly-b19.fasta contig_6:1960041-1970041   > 1.BSA4upstream-c6-1960041-1970041-minimap.fasta
samtools faidx assembly-b19.fasta contig_7:3072313-3082313   > 2.GTDNUdownstream-c7-3072313-3082313-minimap.fasta
samtools faidx assembly-b19.fasta contig_7:3063445-3073445   > 2.GTDNUdownstream-c7-3063445-3073445-minimap.fasta

cat *-minimap.fasta > inserts-minimap.fasta

minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/GCA_006942115.1_ASM694211v1_genomic.fna  inserts-minimap.fasta > aln-b19.sam

grep -v "@SQ" aln-b19.sam | awk '$3!="*"'  | awk '{print $1, $2, $3, $4, $6}'
```
```
contig_6:1960041-1970041 16 PQNB01000001.1 778195 499S39M1I3680M1D5782M
contig_6:1969041-1970041 16 PQNB01000001.1 778195 499S39M1I462M
contig_7:3063445-3073445 16 PQNB01000028.1 440888 2S2305M1D2856M1D1440M1D710M2D2688M
contig_7:3072313-3082313 0 PQNB01000006.1 621765 8473S146M3D1131M1D211M40S
contig_7:3072313-3082313 2064 PQNB01000028.1 440888 8870H1131M
contig_7:3072313-3082313 2064 PQNB01000001.1 333817 8271H184M1D419M1127H
contig_7:3072313-3082313 2048 PQNB01000001.1 339391 2747H4M1D564M6686H
contig_7:3072313-3082313 2048 PQNB01000001.1 1961659 7215H94M1D452M2240H
contig_7:3072313-3082313 2048 PQNB01000006.1 526936 5111H234M1D301M4355H
contig_7:3072313-3082313 2048 PQNB01000006.1 528580 4801H68M1D236M4896H
contig_7:3072313-3082313 2064 PQNB01000022.1 719752 7257H300M2444H
contig_7:3072313-3082313 2064 PQNB01000006.1 626849 2792H150M7059H
contig_7:3072313-3082313 2064 PQNB01000023.1 405895 1571H155M8275H
contig_7:3072445-3073445 16 PQNB01000028.1 440888 2S999M
```
