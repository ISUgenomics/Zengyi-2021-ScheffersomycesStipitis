## Alignments for barcode 15

####  Barcode15 (NC29) alignments:

* Nova : /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment

* Jan 25, 2021

I am aligning all the inserts on the assembly of strain `NC29` to find the location of inserts.


I look at the primary `flag 0` and reverse `flag 16` alignments in the sam files.


```bash
grep -v "@SQ" aln-b15.sam | awk '$3!="*"' | awk '$2==16' | awk '{print $1, $2, $3, $4, $6}'
```
```
6.BNU-loxP 16 contig_6 3435111 3160M1I1768M1I409M
1.BSA4upstream 16 contig_6 3421906 13205M1860S
5.BTDN 16 contig_4 2058782 1637M1I2196M1I4037M1I409M
4.BTDNU-loxP 16 contig_4 2059259 2000S1160M1I2196M1I4037M1I409M
3.GSA4 16 contig_6 3421427 13684M2403S
2.GTDNUdownstream 16 contig_4 2059259 1527S1160M1I2196M1I2374M4D18M5D3M4D8M3D2M2D8M3I4M1D10M3I1M1I14M3I11M2I11M4I5M1I7M1I4M3I1M1D4M2I2M1I10M2D15M1I4M1I8M2I8M3D6M3D9M1D5M1D1M1D3M2I3M4D2M1D2M4D4M1I22M1D6M2I1M2I11M1I11M1D2M2I6M6I2M1D524M2D3M2I182M605S
```

No alignment with flag of `0` (primary alignment) was found.

For this strain we know that `6.BNU-loxP` and `5.BTDN 16 contig_4` are inserted. From the alignment we can see that `6.BNU-loxP` alignment has all the basepairs but 2 matched `3160M1I1768M1I409M`. ( Only 2 I, or inserts from the CIGAR).
For `5.BTDN` the CIGAR is `1637M1I2196M1I4037M1I409M` with only 3 insertions and all other basepairs match. So this is a good alignment.
alignment of other inserts include lots of hard or soft clipping because inserts have several parts in common. SO only some sections of other inserts align to the assembled strain.

I now proceed to run alignments on read sections of the assembly near the insertion area.

I will use two different aligner to compare their results, minimap2 and gmap:

#### Minimap2

For `minimap2` aligner I use 1000 base pair down stream of the insertion location on the assembled genome:

* 5.BTDN insert
from the sam file I know the insert location : contig_4 2058782


```bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b15

cp ../../../02-fly/out_flye_b15/assembly.fasta ./assembly-b15.fasta

module load samtools
samtools faidx assembly-b15.fasta

samtools faidx assembly-b15.fasta contig_4:2057782-2058782 > 5.BTDN-c4-2057782-2058782-minimap.fasta
```

* 6.BNU-loxP
From the sam file I know the location of insert: contig_6 3435111

```bash
samtools faidx assembly-b15.fasta contig_6:3434111-3435111 > 6.BNU-loxP-c4-3434111-3435111-minimap.fasta
```

#### gmap

For `gmap` aligner I use 1000 base pair before the insertion location on the assembled genome plus insert length plus 1000 basepairs after the insert. First I need to know the insert length :

```bash
grep -v ">" BNU-loxP_sequence.fasta | wc | awk '{print $3-$1}'
  5339

grep -v ">" BTDN_sequence.fasta | wc | awk '{print $3-$1}'
  8282
```

* 5.BTDN
from the sam file I know the insert location : contig_4 2058782

2058782+8282+1000=2068064

```bash
samtools faidx assembly-b15.fasta contig_4:2058782-2068064 > 5.BTDN-c4-2058782-2068064-gmap.fasta
```

* 6.BNU-loxP
From the sam file I know the location of insert: contig_6 3435111

3435111+5339+1000=3441450

```bash
samtools faidx assembly-b15.fasta contig_6:3435111-3441450 > 6.BNU-loxP-c6-3435111-3441450-gmap.fasta
```

Concatenating the read sections:

```
cat *-gmap.fasta > inserts-gmap.fasta
cat *-minimap.fasta > inserts-minimap.fasta
```

Adding more details to the inset-IDs:
```
less inserts-gmap.fasta     | sed 's/contig_6/contig_6-6.BNU-loxP/g' | sed 's/contig_4/contig_4-5.BTDN/g'  > inserts-gmap_correctID.fasta

less inserts-minimap.fasta     | sed 's/contig_6/contig_6-6.BNU-loxP/g' | sed 's/contig_4/contig_4-5.BTDN/g'  > inserts-minimap_correctID.fasta
```

Running gmap and minimap2:


*  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/sub/minimap2_b15.sub

```bash
#!/bin/bash
#SBATCH -N 1
#SBATCH --ntasks-per-node=30
#SBATCH -t 48:00:00
#SBATCH -J sub/minimap2_b15
#SBATCH -o ./log/sub/minimap2_b15.o%j  
#SBATCH -e ./log/sub/minimap2_b15.e%j
#SBATCH --mail-user=msayadi@iastate.edu  
#SBATCH --mail-type=begin
#SBATCH --mail-type=end

cd $SLURM_SUBMIT_DIR
ulimit -s unlimited

module load minimap2
minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/GCA_006942115.1_ASM694211v1_genomic.fna /work/gif/M
aryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b15/inserts-minimap_correctID.fasta   > /work/gif/Maryam/projects/Zengyi-2021-Scheffe
rsomycesStipitis/03-alignment/uniq-alignemnt/b15/aln-b15-minimap.sam
scontrol show job
$SLURM_JOB_ID
```

*  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/sub/gmap_b15.sub

```bash
#!/bin/bash
#SBATCH -N 1
#SBATCH --ntasks-per-node=30
#SBATCH -t 48:00:00
#SBATCH -J sub/gmap_b15
#SBATCH -o ./log/sub/gmap_b15.o%j  
#SBATCH -e ./log/sub/gmap_b15.e%j
#SBATCH --mail-user=msayadi@iastate.edu  
#SBATCH --mail-type=begin
#SBATCH --mail-type=end

cd $SLURM_SUBMIT_DIR
ulimit -s unlimited

module load gmap-gsnap
gmap -D /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt  -d ScheffStipDB -B 5 -t 30 --input-buffer-size=1000000 --output-b
uffer-size=1000000 -f samse /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b15/inserts-gmap_correctID.fasta  > /work/gif/
Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b15/b15-gmap.sam
scontrol
show job $SLURM_JOB_ID
```

alignment results from the sam files:

Two alignment found for `6.BNU-loxP` and one for `5.BTDN` by minimap.

```
contig_4-5.BTDN:2057782-2058782	16	PQNB01000005.1	31875	60	1S1000M

contig_6-6.BNU-loxP:3434111-3435111	0	PQNB01000001.1	348838	60	738M263S

contig_6-6.BNU-loxP:3434111-3435111	2064	PQNB01000001.1	1961940	60	1H268M732H
```
gmap finds only one alignment for `6.BNU-loxP`.

```
contig_6-6.BNU-loxP:3435111-3441450	16	PQNB01000001.1	1961659	40	3081S94M1D452M2713S
```
gmap could not find the primary alignment that minimap2 did found for `6.BNU-loxP` .


| insert | gmap location | gmap CIGAR|  minimap location | minimap CIGAR|
| --- | --- | --- | ---| ---|
|5.BTDN | NA |NA| 31875 | 1S1000M |
|6.BNU-loxP  |1961659| 3081S94M1D452M2713S |348838|738M263S|
