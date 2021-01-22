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

For barcode13:

I am trying to look at the aligned regions.

```bash
grep -v "@SQ" aln-b13.sam | awk '$3!="*"' | awk '$2==0' | awk '{print $1, $2, $3, $4}'
PQNB01000006.1 0 BNU-loxP-contig_20:3300-3400 1
PQNB01000007.1 0 BSA4upstream-contig_21:5050-5150 1
PQNB01000008.1 0 BNU-loxP-contig_20:3300-3400 1
PQNB01000020.1 0 GTDNUdownstream-scaffold_3:1-100 1
PQNB01000027.1 0 BNU-loxP-contig_20:3300-3400 1
```

why they are all aligned at position 1?
