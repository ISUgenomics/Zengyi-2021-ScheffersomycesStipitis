# Aligning the inserts to the assembled genomes

* Nova
* Jan 21, 2021

The goal is to align the insert to the assembled genomes and find the location of inserts.

#### Inserts

The format of the insert sequences provided to us is `.dna`. I have to convert them to fasta format first.

The program that generats `.dna` formats is [SnapGene](https://www.snapgene.com
  ). I download a free trial for my local machine and converted all the inserts to `fasta` format.

* location of inserts: `/work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData`

Inserts:

```
BNU-loxP_sequence.fasta
BSA4upstream.fasta
BTDNU-loxP_sequence.fasta  
BTDN_sequence.fasta
GSA4_sequence.fasta
GTDNUdownstream.fasta

```


```
cat *.fasta> inserts-all.fasta
```

#### Running minimap2 to align the inserts to the assembled genomes

I am using [minimap2](https://github.com/lh3/minimap2#general) for alignment.

* run_minimap2.bash
```bash
#!/bin/bash

barcode="$1"

filename=`echo minimap2_b$barcode`
jobname=$filename
filename+=".sub"



echo $filename
echo "file name $filename"

echo "#!/bin/bash" > $filename
echo "#SBATCH -N 1" >> $filename
echo "#SBATCH --ntasks-per-node=30" >> $filename
echo "#SBATCH -t 48:00:00" >>  $filename
echo "#SBATCH -J $jobname " >> $filename
echo "#SBATCH -o ./log/$jobname.o%j  " >> $filename
echo "#SBATCH -e ./log/$jobname.e%j " >> $filename
echo "#SBATCH --mail-user=msayadi@iastate.edu  " >> $filename
echo "#SBATCH --mail-type=begin " >> $filename
echo "#SBATCH --mail-type=end " >> $filename

echo >> $filename

echo "cd \$SLURM_SUBMIT_DIR" >> $filename
echo "ulimit -s unlimited" >> $filename


echo >> $filename

echo "module load minimap2" >> $filename
echo "minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData/inserts-all.fasta  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/02-fly/out_
flye_b$barcode/assembly.fasta  > aln-b$barcode\.sam " >> $filename

echo "scontrol show job \$SLURM_JOB_ID" >> $filename

sbatch $filename

```

```
-L           write CIGAR with >65535 ops at the CG tag
-a           output in the SAM format (PAF by default)
-x STR       preset (always applied before other options; see minimap2.1 for details) []
map-ont : For mapping long noisy genomic reads. `map-ont` uses ordinary minimizers as seeds.
```
Running the script in a for loop:
```bash
for n in 13 14 15 16 17 18 19 ; do  ./run_minimap2.bash $n ; done
```

The alignment is done very quickly.

Now I want to check which insert is in each assembly.

```bash
grep -v "@SQ" aln-b13.sam | awk '$3!="*"' | awk '$2==0' | awk '{print $3}' | sort | uniq -c
      1 1.BSA4upstream
      1 2.GTDNUdownstream
      1 6.BNU-loxP

```
So there are 3 inserts in barcode13 assembly. Let me find where they are.

```bash
grep -v "@SQ" aln-b13.sam | awk '$3!="*"' | awk '$2==0' | awk '{print $1, $2, $3, $4}'

  contig_20 0 6.BNU-loxP 3339
  contig_21 0 1.BSA4upstream 5100
  scaffold_3 0 2.GTDNUdownstream 1
```


I am going to try to extract a region of the assembly that I want

example:

```bash
samtools faidx genome.fa
samtools faidx genome.fa contig:pos1-pos2
```

making a new directory for the reads +50bp and -50bp of each insert:

* Samtools didn't work with soft link and I had to copy the assembly to the working directory!

```bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/
mkdir uniq-alignment

module laod samtools
cp ../../../02-fly/out_flye_b13/assembly.fasta ./assembly-b13.fasta

samtools --indexing faixd assembly-b13.fasta

samtools faidx assembly-b13.fasta contig_20:3300-3400
samtools faidx assembly-b13.fasta contig_21:5050-5150 > BSA4upstream-5050-5150.fasta
samtools faidx assembly-b13.fasta scaffold_3:1-100 > GTDNUdownstream-1-100.fasta

```

Now I have the reads around the insertion area I will align these reads back to the Scheffersomyces Stipitis to have the same reference for all the inserts in all the barcodes.

First downloading the [Scheffersomyces stipitis genome](https://www.ncbi.nlm.nih.gov/genome/?term=Scheffersomyces%20stipitis[Organism]&cmd=DetailsSearch).


Last version [Index of /genomes/genbank/fungi/Scheffersomyces_stipitis/latest_assembly_versions/GCA_006942115.1_ASM694211v1](https://ftp.ncbi.nlm.nih.gov/genomes/genbank/fungi/Scheffersomyces_stipitis/latest_assembly_versions/GCA_006942115.1_ASM694211v1) last modified at 2019-07-19

```
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt
wget https://ftp.ncbi.nlm.nih.gov/genomes/genbank/fungi/Scheffersomyces_stipitis/latest_assembly_versions/GCA_006942115.1_ASM694211v1/GCA_006942115.1_ASM694211v1_genomic.fna.gz
gunzip GCA_006942115.1_ASM694211v1_genomic.fna.gz

```
 Now I am going to align all the read sections I chose to this genome.

First concatenate the reads and changing their IDs so it is easier to understand after alignment.

 ```
 cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b13

 less all-insert-regions.fasta        | sed 's/contig_20:3300-3400/BNU-loxP-contig_20:3300-3400/g' | sed 's/contig_21:5050-5150/BSA4upstream-contig_21:5050-5150/g' |s
ed 's/scaffold_3:1-100/GTDNUdownstream-scaffold_3:1-100 /g'  > all-insert-regions-correctIDs.fasta
```

* more run_minimap2.bash
```bash
#!/bin/bash

barcode="$1"

filename=`echo minimap2_b$barcode`
jobname=$filename
filename+=".sub"



echo $filename
echo "file name $filename"

echo "#!/bin/bash" > $filename
echo "#SBATCH -N 1" >> $filename
echo "#SBATCH --ntasks-per-node=30" >> $filename
echo "#SBATCH -t 48:00:00" >>  $filename
echo "#SBATCH -J $jobname " >> $filename
echo "#SBATCH -o ./log/$jobname.o%j  " >> $filename
echo "#SBATCH -e ./log/$jobname.e%j " >> $filename
echo "#SBATCH --mail-user=msayadi@iastate.edu  " >> $filename
echo "#SBATCH --mail-type=begin " >> $filename
echo "#SBATCH --mail-type=end " >> $filename

echo >> $filename

echo "cd \$SLURM_SUBMIT_DIR" >> $filename
echo "ulimit -s unlimited" >> $filename


echo >> $filename

echo "module load minimap2" >> $filename
echo "minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b$barcode/all-insert-regions-correctIDs.fasta
 /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/GCA_006942115.1_ASM694211v1_genomic.fna > /work/gif/Maryam/projects/Zengyi
-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b$barcode/aln-b$barcode.sam " >> $filename

echo "scontrol show job \$SLURM_JOB_ID" >> $filename

sbatch $filename
```

running in a for loop.

```bash
for n in 13 14 15 16 17 18 19


```

####  Barcode13 (strain 6):

I am trying to look at the aligned regions.

```bash
grep -v "@SQ" aln-b13.sam | awk '$3!="*"' | awk '$2==0' | awk '{print $1, $2, $3, $4}'
PQNB01000006.1 0 BNU-loxP-contig_20:3300-3400 1
PQNB01000007.1 0 BSA4upstream-contig_21:5050-5150 1
PQNB01000008.1 0 BNU-loxP-contig_20:3300-3400 1
PQNB01000020.1 0 GTDNUdownstream-scaffold_3:1-100 1
PQNB01000027.1 0 BNU-loxP-contig_20:3300-3400 1
```
I figured that I have mapped the genome to the reads! I have to switch the genome and reads:

```bash
minimap2 -aLx map-ont  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/02-fly/out_flye_b$barcode/assembly.fasta /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/
00-RawData/inserts-all.fasta   > aln-b$barcode\.sam "
```
I repeated the alignment and got the new sam files.
Now I look at the primary `flag 0` and reverse `flag 16` alignments in the sam files.


```bash
grep -v "@SQ" aln-b13.sam | awk '$3!="*"' | awk '$2==16' | awk '{print $1, $2, $3, $4, $6}'

  1.BSA4upstream 16 contig_18 784739 15065M
  3.GSA4 16 contig_18 784260 13972M4D18M5D3M4D8M3D2M2D8M3I4M1D10M3I1M1I14M3I11M2I11M4I5M1I7M1I4M3I1M1D4M2I2M1I10M2D15M1I4M1I8M2I8M3D6M3D9M1D5M1D1M1D3M2I3M4D2M1D2M4D4M1I22M1D6M2I1M2I11M1I11M1D2M2I6M6I2M1D524M2D3M2I182M1106S
```

and
```#!/usr/bin/env bash
grep -v "@SQ" aln-b13.sam | awk '$3!="*"' | awk '$2==0' | awk '{print $1, $2, $3, $4, $6}'
  6.BNU-loxP 0 scaffold_3 1115711 2078S95M1I2688M477S
  5.BTDN 0 scaffold_3 1110233 1065S182M2D2M2I522M4D7M2D17M1D11M3D2M1D7M1I16M1D10M4I2M1I2M4I3M2D3M1I1M1I4M1I7M7I3M1D12M2D7M1D5M1D16M2I7M1D5M2D4M1I1M3D4M1D4M1D7M4D11M2D2M1D6M3I9M4D2M5D3M1I8M1D6M1D10M2I1M3I8M4I4M5I18M4I2124M1I2441M1I1165M477S
  4.BTDNU-loxP 0 scaffold_3 1110233 1065S182M2D2M2I522M4D7M2D17M1D11M3D2M1D7M1I16M1D10M4I2M1I2M4I3M2D3M1I1M1I4M1I7M7I3M1D12M2D7M1D5M1D16M2I7M1D5M2D4M1I1M3D4M1D4M1D7M4D11M2D2M1D6M3I9M4D2M5D3M1I8M1D6M1D10M2I1M3I8M4I4M5I18M4I2124M1I2441M1I2688M477S
  2.GTDNUdownstream 0 scaffold_3 1109630 407M1I73M1I1060M1I2195M1I2441M1I2688M4S

```

I am expecting that `1.BSA4upstream` and `2.GTDNUdownstream` have been inserted in this strain.

From above alignment I can see that `  1.BSA4upstream 16 contig_18 784739 15065M` is a perfect match.

For `2.GTDNUdownstream` I think this is the best alignment `2.GTDNUdownstream 0 scaffold_3 1109630 407M1I73M1I1060M1I2195M1I2441M1I2688M4S`. It is not perfect however. It has `5` insertions and `4` soft clipping at the end which I think is still acceptable.

I now proceed to run alignments on read sections of the assembly near the insertion area.

First I use minimap to find alignemnt of a section of the assembly from  x-1000 bp till x bp.

* 1.BSA4upstream (x=contig_18 784739)

```#!/usr/bin/env bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b13
module load samtools

samtools --indexing faixd assembly-b13.fasta

samtools faidx assembly-b13.fasta contig_18:783739-784739 > BSA4upstream-c18-783739-784739-minimap.fasta


```
Then `gmap` to align the section between x-1000 and x+insert-lenght+1000

insert length = 15065 bp

```bash
samtools faidx assembly-b13.fasta contig_18:783739-800804 > BSA4upstream-c18-783739-800804-gmap.fasta
```
784739+ 15065 + 1000 = 800804



* 2.GTDNUdownstream

2.GTDNUdownstream 0 scaffold_3 1109630 407M1I73M1I1060M1I2195M1I2441M1I2688M4S

```
samtools faidx assembly-b13.fasta scaffold_3:1108630-1109630 > GTDNUdownstream-s3-1108630-1109630-minimap.fasta
```

and for gmap
```bash
samtools faidx assembly-b13.fasta scaffold_3:1108630-1116503 > GTDNUdownstream-s3-1108630-1116503-gmap.fasta

```
insert lenght = 8877
1109630 + 8873 + 1000 = 1119503

Concatenating the read sections:
```
cat *-gmap.fasta > inserts-gmap.fasta
cat *-minimap.fasta > inserts-minimap.fasta
```
#### Running alignments

changing the IDs:
```#!/usr/bin/env bash
less inserts-minimap.fasta | sed 's/scaffold_3/scaffold_3-2.GTDNUdownstream/g' | sed 's/contig_18/contig_18-1.BSA4upstream/g'> inserts-minimap_correctID.fasta

less inserts-gmap.fasta | sed 's/scaffold_3/scaffold_3-2.GTDNUdownstream/g' | sed 's/contig_18/contig_18-1.BSA4upstream/g' > inserts-gmap_correctID.fasta

```
* running minimap :

```#!/usr/bin/env bash

```

* running gmap :
Building genome database
```
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt

gmap_build -d ScheffStipDB  -D /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt  /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/GCA_006942115.1_ASM694211v1_genomic.fna
```

running gmap

* run_gmap.bash
```
#!/bin/bash

barcode="$1"

filename=`echo gmap_b$barcode`
jobname=sub/$filename
filename+=".sub"



echo sub/$filename
echo "file name sub/$filename"

echo "#!/bin/bash" > sub/$filename
echo "#SBATCH -N 1" >> sub/$filename
echo "#SBATCH --ntasks-per-node=30" >> sub/$filename
echo "#SBATCH -t 48:00:00" >>  sub/$filename
echo "#SBATCH -J $jobname " >> sub/$filename
echo "#SBATCH -o ./log/$jobname.o%j  " >> sub/$filename
echo "#SBATCH -e ./log/$jobname.e%j " >> sub/$filename
echo "#SBATCH --mail-user=msayadi@iastate.edu  " >> sub/$filename
echo "#SBATCH --mail-type=begin " >> sub/$filename
echo "#SBATCH --mail-type=end " >> sub/$filename

echo >> sub/$filename

echo "cd \$SLURM_SUBMIT_DIR" >> sub/$filename
echo "ulimit -s unlimited" >> sub/$filename


echo >> sub/$filename

echo "module load gmap-gsnap" >> sub/$filename
echo "gmap -D /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt  -d ScheffStipDB -B 5 -t 30 --input-buffer-size=1000000 --output-buffer-size=1000000 -f gf
f3_gene /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b$barcode/inserts-gmap_correctID.fasta  > /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesS
tipitis/03-alignment/uniq-alignemnt/b$barcode/b$barcode-gmap.gff3" >> sub/$filename

echo "scontrol show job \$SLURM_JOB_ID" >> sub/$filename

sbatch sub/$filename
```

and run it in a loop or as
```
./run_gmap.bash 13
```

looking at the `gff3` output :

```
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/03-alignment/uniq-alignemnt/b13

less b13-gmap.gff3 | grep -v "#"| awk '{print $9}' | sed 's/:/ /g' | awk '{print $1}' | sort | uniq
```
```
ID=scaffold_3-2.GTDNUdownstream
```

gmap probably did only picked `GTDNUdownstream` insert because it is more used for long less noisy reads and more sensitive to mismatches. minimap2 is designed for shorter more noisy reads.

| insert | gmap location | gmap CIGAR|  minimap location | minimap CIGAR|
| --- | --- | --- | ---| ---|
|1.BSA4upstream  | NA |NA |777685 |517M484S |
|2.GTDNUdownstream |333817 |!!! | 333817| 395S112M1D71M1D414M9S| 
-----
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

alignment results:

| insert | gmap location | gmap CIGAR|  minimap location | minimap CIGAR|
| --- | --- | --- | ---| ---|
| 4.BTDNU-loxP | 199853 |!!!| 629073 |17M1D984M |
| 3.GSA4 | NA | NA | 2089204|561M440S|
