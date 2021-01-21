# Assembly with Flye genome assembler
Goal is finding the insert locations after finishing the assemblies.

* Nova: /work/gif/Maryam/Programfiles/Flye
* Jan18, 2020

## Install Flye

https://github.com/fenderglass/Flye/blob/flye/docs/INSTALL.md

```
git clone https://github.com/fenderglass/Flye
cd flye
make

```

* Version
```
./flye --version
2.8.2-b1691
```

## Running Flye

* fly_b13.sub

```bash
#!/bin/bash
#SBATCH -N 1
#SBATCH --ntasks-per-node=30
#SBATCH -p fat
#SBATCH -t 168:00:00
#SBATCH -J fly-6
#SBATCH -o log/fly-6.o%j
#SBATCH -e log/fly-6.e%j
#SBATCH --mail-user=msayadi@iastate.edu
#SBATCH --mail-type=begin
#SBATCH --mail-type=end
cd $SLURM_SUBMIT_DIR
ulimit -s unlimited


flye --nano-raw /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/01-QC/output-barcode13/trimmedReads/FAO68114_pass_barcode13_598c81f2_0_adaptersRemoved.fastq --out-dir out_barcode13
 --genome-size 15m --threads 30 -i 4

```

Flye log complained it had a duplicate read. Andrew has reported the same issue before as dosumented [here](https://github.com/ISUgenomics/SWFSC_2020_WhiteAbalone/blob/6ea3216d38c7b31212209a6e2fa2e9915a732bec/2019_WhiteAbalone/notebook_severin/condo/WhiteAbalone2018/07_flye.md). He has removed the duplicates.

In my case :
```
grep "Barcode-13-(forward)_26a4210c-f10d-46ae-b345-ea93f90e14ff"
```
` FAO68114_pass_barcode13_598c81f2_0_adaptersRemoved.fastq
@Barcode-13-(forward)_26a4210c-f10d-46ae-b345-ea93f90e14ff runid=598c81f298f24cf974f62bf9136b2ea7dfc37f41 read=541 ch=195 start_time=2020-12-22T20:40:40Z flow_cell_id=FAO68114 protocol_group_id=1585 sample_id=no_sample barcode=barcode13 barcode_alias=barcode13_(left)
@Barcode-13-(forward)_26a4210c-f10d-46ae-b345-ea93f90e14ff runid=598c81f298f24cf974f62bf9136b2ea7dfc37f41 read=541 ch=195 start_time=2020-12-22T20:40:40Z flow_cell_id=FAO68114 protocol_group_id=1585 sample_id=no_sample barcode=barcode13 barcode_alias=barcode13_(right)`

Checking to see how many repeated IDs I have:
```
awk '(NR%4==1){print}' FAO68114_pass_barcode13_598c81f2_0_adaptersRemoved.fastq | sort -S 2G | awk '{print $1}' | uniq -d
```

`@Barcode-13-(forward)_26a4210c-f10d-46ae-b345-ea93f90e14ff`

Ok, it is only one. I did check and the reads are different. Only the ids are the same. It seems that one is left one is right. I am not sure what it means but I decided to rename them instead of deleting the read!

```bash
less "FAO68114_pass_barcode13_598c81f2_0_adaptersRemoved.fastq" | tr '@Barcode-13-(forward)_26a4210c-f10d-46ae-b345-ea93f90e14ff runid=598c81f298f24cf974f62bf9136b2ea7dfc37f41 read=541 ch=195 start_time=2020-12-22T20:40:40Z flow_cell_id=FAO68114 protocol_group_id=1585 sample_id=no_sample barcode=barcode13 barcode_alias=barcode13_(left)' '@Barcode-13-(forward)_26a4210c-f10d-46ae-b345-ea93f90e14ff_l runid=598c81f298f24cf974f62bf9136b2ea7dfc37f41 read=541 ch=195 start_time=2020-12-22T20:40:40Z flow_cell_id=FAO68114 protocol_group_id=1585 sample_id=no_sample barcode=barcode13 barcode_alias=barcode13_(left)'
```

And restart. It is running.

Now I need to do the same for the next 6 samples.

* run_flye.bash
```bash
#!/bin/bash

barcode="$1"

filename=`echo flye_b$barcode`
jobname=$filename
filename+=".sub"



echo $filename
echo "file name $filename"

echo "#!/bin/bash" > $filename
echo "#SBATCH -N 1" >> $filename
echo "#SBATCH --ntasks-per-node=30" >> $filename
echo "#SBATCH -t 168:00:00" >>  $filename
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

echo "flye --nano-raw /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/01-QC/output-barcode$barcode/trimmedReads/FAO68114_pass_barcode$barcode\_598c81f2_0_adaptersRemoved.fastq --ou
t-dir out_$jobname --genome-size 15m --threads 30 -i 4" >> $filename

echo "scontrol show job \$SLURM_JOB_ID" >> $filename

sbatch $filename

```

and run it true a loop :

```bash
for n in 14 15 16 17 18 19 ; do  ./run_flye.bash $n ; done

```

I have the same issue with some other barcodes:

* barcode-14
```bash
awk '(NR%4==1){print}'
 FAO68114_pass_barcode14_598c81f2_0_adaptersRemoved.fastq | sort -S 2G | awk '{print $1}' | uniq -d
  ```


`@Barcode-14-(forward)_15f3edea-9434-4c3e-9e05-1e06574fd024
@Barcode-14-(forward)_50d3b604-43e6-4244-b829-7a61e76a9c3d
@Barcode-14-(forward)_94cd6aeb-0dd5-42aa-8e6a-5742ab66ffa2
`

* barcode-16
```bash
awk '(NR%4==1){print}'
 FAO68114_pass_barcode16_598c81f2_0_adaptersRemoved.fastq | sort -S 2G | awk '{print $1}' | uniq -d
  ```

`@SQK-NSK007-Y_1af3ca89-a7fd-4429-8d36-994014724aee
`

* barcode-19
```bash
awk '(NR%4==1){print}'
 FAO68114_pass_barcode19_598c81f2_0_adaptersRemoved.fastq | sort -S 2G | awk '{print $1}' | uniq -d
  ```

`
@Barcode-19-(forward)_11bc1ec7-78c9-43e8-ac4c-bb39e41887df
`

renamed all the duplicate IDs. ( added `-l` at the end of the ids).

All of the assemblies finished very quickly.

#### Assembly Stats


```bash
module load gcc/7.3.0-xegsmw4
module load nextflow
module load singularity

../../../Programfiles/assemblyStats/main.nf --listDatasets -profile singularity
```

* disk space error
` FATAL:   While making image from oci registry: while building SIF from layers: conveyor failed to get: Error initializing source oci:/home/msayadi/.singularity/cache/oci:43e181f6c95958eb5c639526bfc3330844a9b45c021d551e27dd1906c51f218c: Error writing blob: open /home/msayadi/.singularity/cache/oci/oci-put-blob578989333: disk quota exceeded
`

I tried to redirect the cache to a temp directory but didn't work..

```bash
SINGULARITY_LOCALCACHEDIR="/work/gif/Maryam/dot-files/"
```
still had the same problem. SO I decided to delete the content of cache directory to free up space!

```bash
du -hs /home/msayadi/
```
`
.singularity/cache/oci
2.1G	/home/msayadi/.singularity/cache/oci
`
```bash
rm  -rf /home/msayadi/.singularity/cache/oci/*
```

Ok, it works and now I have a list of species I can choose for Bosco.

I chose `eukaryota_odb10` from the list. One group down is `fungi_odb10`.


I am trying this first :

```
nextflow run /work/gif/Maryam/Programfiles/assemblyStats/main.nf --genome out_barcode13/assembly.fasta --outdir stats-out-b13  --options "-l  fungi_odb10"  -profile singularity,nova
```

* Error:

`N E X T F L O W  ~  version 20.07.1
Launching "/work/gif/Maryam/Programfiles/assemblyStats/main.nf" [astonishing_stone] - revision: 4565f79756
Missing "fromPath" parameter`


That was a silly mistake! `--genome` should be `--genomes`.

Running again:
```bash
nextflow run /work/gif/Maryam/Programfiles/assemblyStats/main.nf --genomes out_barcode13/assembly.fasta --outdir stats-out-b13  --options "-l  fungi_odb10"  -profile singularity,nova
```

disk space error again! I should be able to change the cache directory!

changing cache directory didn't work. So I cleared cache and ran again.

This time I got Bosco error:


`Command error:
  ERROR:	Augustus did not recognize any genes matching the dataset fungi_odb10 in the input file. If this is unexpected, check your input file and your installation of Augustus
  ERROR:	BUSCO analysis failed !`


###  Starting new Flye assembly with correctly trimmed reads

After reads are properly trimmed with downpore, I am going to check how many reads have been split.

```
wc combine-barcode13_adaptersRemoved.fastq
    625772    2033759 2810117994 combine-barcode13_adaptersRemoved.fastq
```
or `156443` reads for barcode13 after trimming. number of reads before trimming is `156421`.

* FLYE Error:
`ERROR: The input contain reads with duplicated IDs. Make sure all reads have unique IDs and restart. The
 first problematic ID was: Barcode-13-(forward)_e70940d2-cd0a-461e-b4f1-13c2e8bfa03c
`
renaming the reads:
 ```
 grep "Barcode-13-(forward)_e70940d2-cd0a-461e-b4f1-13c2e8bfa03c" combine-barcode13_adaptersRemoved.fastq
@Barcode-13-(forward)_e70940d2-cd0a-461e-b4f1-13c2e8bfa03c runid=598c81f298f24cf974f62bf9136b2ea7dfc37f41 read=11326 ch=222 start_time=2020-12-23T07:36:02Z flow_cell_id=FAO68114 protocol_group_id=1585 sample_id=no_sample barcode=barcode13 barcode_alias=barcode13_(left)
@Barcode-13-(forward)_e70940d2-cd0a-461e-b4f1-13c2e8bfa03c runid=598c81f298f24cf974f62bf9136b2ea7dfc37f41 read=11326 ch=222 start_time=2020-12-23T07:36:02Z flow_cell_id=FAO68114 protocol_group_id=1585 sample_id=no_sample barcode=barcode13 barcode_alias=barcode13_(right)
```

renames read  `@Barcode-13-(forward)_e70940d2-cd0a-461e-b4f1-13c2e8bfa03c_right`
 to `@Barcode-13-(forward)_e70940d2-cd0a-461e-b4f1-13c2e8bfa03c-r`.

next error:
`ERROR: The input contain reads with duplicated IDs. Make sure all reads have unique IDs and restart. The
 first problematic ID was: Barcode-13-(forward)_7879f894-d138-4fd2-b703-27d5c73ad7f8`



It seams that downpore assigns the same IDs ( with a slight difference) to the split reads and that causes error in Flye.

For example for barcode 13:
```
awk '(NR%4==1){print}' combine-barcode13.fastq | sort -S 2G | awk '{print $1}' | uniq -d
@Barcode-13-(forward)_0540bc80-df56-4d88-8354-79beebb08759
@Barcode-13-(forward)_07f72186-831d-4a3e-850b-91176dacea31
@Barcode-13-(forward)_0907c057-f317-452b-a66d-654ad1a98bec
@Barcode-13-(forward)_172c1b13-bcae-4c72-a9ab-a4a40aca967e
@Barcode-13-(forward)_18ccf23b-5819-4e96-b739-c5d4c60fa1d4
@Barcode-13-(forward)_19a9256e-854a-4d81-af66-524741811098
@Barcode-13-(forward)_1e27661c-1178-4e2b-adf1-64bd5c70f99f
@Barcode-13-(forward)_253acfa7-3e1f-4d9a-ab6a-e0652e6f9aff
@Barcode-13-(forward)_3204c58b-4cf8-4137-9f77-6b3d049993e2
@Barcode-13-(forward)_49fd47b7-6862-4e67-be20-d07259a59b79
@Barcode-13-(forward)_522f49d9-43ab-4a21-aef0-d8df698bf846
@Barcode-13-(forward)_7879f894-d138-4fd2-b703-27d5c73ad7f8
@Barcode-13-(forward)_869d1341-d94d-4c99-aa1e-1da8417e3821
@Barcode-13-(forward)_880a24fb-cd4d-40cc-a899-efe3962f74b5
@Barcode-13-(forward)_934c0678-3ddb-465c-90c1-b36d68e5424d
@Barcode-13-(forward)_a718823e-b72f-47c3-9956-e2ba097dfd50
@Barcode-13-(forward)_aa4cf1ad-f909-453d-bd97-dd2b8920da6a
@Barcode-13-(forward)_b10ee1eb-11cc-423c-a946-85d1a8a62b4c
@Barcode-13-(forward)_d1ff792d-8eca-4447-a631-a0da32f7dc2e
@Barcode-13-(forward)_fb4b09e9-da64-46be-97be-e79d08ee8978
@e7a950db-ded2-4392-a565-1d8ef0f7a6c2
```

I looked for one of these duplicate ids for example :

```
grep "@Barcode-13-(forward)_0540bc80-df56-4d88-8354-79beebb08759" combine-barcode13_adaptersRemoved-renameDupls.fastq
```
```
@Barcode-13-(forward)_0540bc80-df56-4d88-8354-79beebb08759 runid=598c81f298f24cf974f62bf9136b2ea7dfc37f41 read=23369 ch=173 start_time=2020-12-23T12:53:48Z flow_cell_id=FAO68114 protocol_group_id=1585 sample_id=no_sample barcode=barcode13 barcode_alias=barcode13_(right)

@Barcode-13-(forward)_0540bc80-df56-4d88-8354-79beebb08759 runid=598c81f298f24cf974f62bf9136b2ea7dfc37f41 read=23369 ch=173 start_time=2020-12-23T12:53:48Z flow_cell_id=FAO68114 protocol_group_id=1585 sample_id=no_sample barcode=barcode13 barcode_alias=barcode13_(left)

```



I am going to add line number to all the ids to make them all uniq.

```bash
paste - - - - < combine-barcode13_adaptersRemoved.fastq | awk 'BEGIN{NF = OFS = "\t"} { print $1"_"NR,$11,$12,$13}' | tr '\t' '\n' > combine-barcode13_uniqIDs.fastq
```

And try flye again. FLye os working.

Repeat that for all the barcodes:

```bash
cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData/1585/1585/1585/20201222_2025_X1_FAO68114_96355fd8/fastq_pass/barcode14
paste - - - - < combine-barcode14_adaptersRemoved.fastq | awk 'BEGIN{NF = OFS = "\t"} { print $1"_"NR,$11,$12,$13}' | tr '\t' '\n' > combine-barcode14_uniqIDs.fastq

cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData/1585/1585/1585/20201222_2025_X1_FAO68114_96355fd8/fastq_pass/barcode15
paste - - - - < combine-barcode15_adaptersRemoved.fastq | awk 'BEGIN{NF = OFS = "\t"} { print $1"_"NR,$11,$12,$13}' | tr '\t' '\n' > combine-barcode15_uniqIDs.fastq

cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData/1585/1585/1585/20201222_2025_X1_FAO68114_96355fd8/fastq_pass/barcode16
paste - - - - < combine-barcode16_adaptersRemoved.fastq | awk 'BEGIN{NF = OFS = "\t"} { print $1"_"NR,$11,$12,$13}' | tr '\t' '\n' > combine-barcode16_uniqIDs.fastq

cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData/1585/1585/1585/20201222_2025_X1_FAO68114_96355fd8/fastq_pass/barcode17
paste - - - - < combine-barcode17_adaptersRemoved.fastq | awk 'BEGIN{NF = OFS = "\t"} { print $1"_"NR,$11,$12,$13}' | tr '\t' '\n' > combine-barcode17_uniqIDs.fastq

cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData/1585/1585/1585/20201222_2025_X1_FAO68114_96355fd8/fastq_pass/barcode18
paste - - - - < combine-barcode18_adaptersRemoved.fastq | awk 'BEGIN{NF = OFS = "\t"} { print $1"_"NR,$11,$12,$13}' | tr '\t' '\n' > combine-barcode18_uniqIDs.fastq

cd /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/00-RawData/1585/1585/1585/20201222_2025_X1_FAO68114_96355fd8/fastq_pass/barcode19
paste - - - - < combine-barcode19_adaptersRemoved.fastq | awk 'BEGIN{NF = OFS = "\t"} { print $1"_"NR,$11,$12,$13}' | tr '\t' '\n' > combine-barcode19_uniqIDs.fastq
```
and run Flye ..

running `run_flye.bash` in a for loop:

```
for n in 14 15 16 17 18 19 ; do  ./run_flye.bash $n ; done
```
* run_flye.bash
```bash
#!/bin/bash

barcode="$1"

filename=`echo flye_b$barcode`
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

echo "flye --nano-raw /work/gif/Maryam/projects/Zengyi-2021-ScheffersomycesStipitis/01-QC/output-barcode$barcode/trimmedReads/combine-barcode$barcode\_uniqIDs.fastq --out-dir out_$jobname --geno
me-size 15m --threads 30 -i 4" >> $filename

echo "scontrol show job \$SLURM_JOB_ID" >> $filename

sbatch $filename
```
All the jobs finished overnight.
Now Lets get some stats:


Renaming the barcode13 folder to be consistent with others:
```
mv out_barcode13/ out_flye_b13
```

```bash
module load gcc/7.3.0-xegsmw4
module load nextflow

module load singularity

nextflow run /work/gif/Maryam/Programfiles/assemblyStats/main.nf --genomes out_flye_b13/assembly.fasta --outdir stats-out-b13  --options "-l  fungi_odb10"  -profile singularity,nova

```
