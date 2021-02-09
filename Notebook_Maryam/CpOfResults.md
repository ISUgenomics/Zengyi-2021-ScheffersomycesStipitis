# Results

## Table 1 , Estimating the insertion locations from the alignment


| strain | insert | position of the insert on the reference genome -10k | position of the insert on the reference genome |position of the insert on the assembly  | confirmed by alignment of inserts on the assembly | confirmed by alignment of reads containing insert to the ref genome (hist)| confirmed by dotplot |
| --- | --- | --- | --- | --- | --- |---| --- |
| barcode 13 (strain 6)| 1.BSA4upstream |PQNB01000001.1 768685|PQNB01000001.1 778685  | contig_18 784739|yes|yes| yes|
| barcode 13 (strain 6)| 2.GTDNUdownstream |PQNB01000023.1  89841|PQNB01000023.1  99841|scaffold_3 1101573 |yes |yes| yes|
| bbarcode 13 (strain 6)| 2.GTDNUdownstream (second location)|PQNB01000023.1 97903|PQNB01000023.1 107903|scaffold_3 1109630 |yes |yes | yes|
|  | | | | | |
| barcode 14 (strain NC-22)| 3.GSA4 |PQNB01000019.1 2080198|PQNB01000019.1 2090198 |contig_6 3421393 |yes |yes | yes|
| barcode 14 (strain NC-22) | 4.BTDNU-loxP |PQNB01000005.1 629073 |PQNB01000005.1 639073|contig_4 123838| yes | yes |yes |
| barcode 14 (strain NC-22) | 4.BTDNU-loxP(second location) | PQNB01000005.1 31875| PQNB01000005.1 41875|contig_4 731511| yes | yes |yes  |
| | | |  || |
| barcode 15 (strain N29) | 5.BTDN |PQNB01000005.1 31875|PQNB01000005.1 41875 |contig_4 2058782| yes| yes| No |
| barcode 15 (strain N29) | 6.BNU-loxP | PQNB01000008.1 239895|PQNB01000008.1 249895|contig_6 3435111|yes |yes |No(almost 5*insert length)|
|  | | | | | |
| barcode 16 (strain 53)| 1.BSA4upstream | PQNB01000001.1 778195|PQNB01000001.1 788195|contig_7 1979461| yes |yes| No|
|barcode 16 (strain 53)  | 2.GTDNUdownstream | PQNB01000019.1 593474 |PQNB01000019.1 603474| contig_5 1589947|yes |yes|No|
|  | | | | | |
| barcode 17 (strain 55) | 1.BSA4upstream |PQNB01000001.1 768685 |PQNB01000001.1 778685| contig_25 784604| yes| yes|yes |
| barcode 17 (strain 55) | 2.GTDNUdownstream  | PQNB01000019.1 1060119|PQNB01000019.1 1070119|contig_27 1123601| yes|yes |yes |
| barcode 17 (strain 55) | 2.GTDNUdownstream  (second location back to back) |PQNB01000019.1 1060119 |PQNB01000019.1 1070119 ||yes|yes |yes |
|  | | | | | |
| barcode 18 (strain 59) | 1.BSA4upstream |PQNB01000001.1 768685|PQNB01000001.1 778685|contig_9 784693 | yes | yes | yes|
| barcode 18 (strain 59) |2.GTDNUdownstream  | PQNB01000022.1 605582|PQNB01000022.1 615582 |scaffold_10 896527| yes | yes | yes|
| barcode 18 (strain 59) |2.GTDNUdownstream (second location back to back)  | PQNB01000022.1 614452	|PQNB01000022.1 624452 |scaffold_10 905392| yes | yes | yes|
|  | | | | | |
| barcode 19 (strain 88) | 1.BSA4upstream | PQNB01000001.1 778195|PQNB01000001.1 788195|contig_6 1970041 |yes |yes |No |
| barcode 19 (strain 88) |2.GTDNUdownstream |PQNB01000028.1 440888 |PQNB01000028.1 450888|contig_7 3073445|yes |yes |No|



## Table 2 , Estimating the insertion locations from the dotpots

| strain | insert | position of the insert on the reference genome | Start position on assembly | End position on Assembly|Multiple of insert length (approx length)|
| --- | --- | --- | --- | --- | ---|
| 6(Barcode 13)| 1.BSA4upstream |*PQNB01000001.1*:778189|*contig_18*:784125|*contig_18*:800119| 1x|
|6(Barcode 13)|2.GTDNUDownstream|*PQNB01000023.1*:99846|*scaffold_3*:1101388|*scaffold_3*:1118291|2x|
|53(barcode16)|1.BSA4upstream|*PQNB01000001.1*:1390811|*contig_7*:1978839|*contig_7*:1994483|1x|
|53(barcode16)|2.GTDNUdownstream|*PQNB01000019.1*:1590524|*contig_5*:1598893|*contig_5*:1598433|1x|
|55(barcode17)|1.BSA4upstream|*PQNB01000001.1*:778203|*contig_25*:784111|*contig_25*:800106|1x|
|55(barcode17)|2.GTDNUdownstream|*PQNB01000019.1*:1123859|*contig_27*:784111|*contig_27*:800106|2x ()|
|59(barcode18)|1.BSA4upstream|*PQNB01000001.1*:778203|*contig_9*:784170|*contig_9*:800219|1x|
|59(barcode18)|2.GTDNUdownstream|*PQNB01000022.1*:615600|*scaffold_10*:896332|*cscaffold_10*:914386|2x()|
|88(barcode19)|1.BSA4upstream|*PQNB01000001.1*:1390849|*contig_6*:1969355|*contig_6*:1985366|1x|
|88(barcode19)|1.BSA4upstream|*PQNB01000001.1*:161184|*contig_6*:733370|*contig_6*:739432|Truncated (6000ish)|
88(barcode19)|2.GTDNUdownstream|*PQNB01000028.1*:471949|*contig_7*:3073264|*contig_7*:3090870|2x|
|             |           |     |            |            |
|NC22(barcode14)|3.GSA4|*PQNB01000019.1*:2089761|*contig_6*:3420846| *contig_6*:3440461| 1x|
|NC22(barcode14)|4.BTDNU-loxP|*PQNB01000005.1*:54430|*contig_4*:123811| *contig_4*:133607| 1x (First location)|
|NC22(barcode14)|4.BTDNU-loxP|*PQNB01000005.1*:651648|*contig_4*:731485| *contig_4*:741113| 1x (second location)|
|N29(barcode15)|5. BTDN|*PQNB01000005.1*:651628|*contig_4*:2058806| *contig_4*:2067054| 1x|
|N29(barcode15)|5. BTDN|*PQNB01000001.1*:161193|*contig_4*:741832| *contig_4*:747898| Truncated? approx 6500|
|N29(barcode15)|6. BNU-loxp|*PQNB01000019.1*:2089761|*contig_6*:3420928| *contig_6*:3445796| 5x (24868)|



| strain | insert | 10kb upstream location 2019 ref | direction of insert |5 kb upstream location 2019 ref | estimated insert location |
| --- | --- | --- | --- | --- | ---|
| 6 (b13) | 1.BSA4upstream | PQNB01000001.1 768684:778196 | + |PQNB01000001.1 773690:778196 |
||||||||
| 53 (b16)| 1.BSA4upstream | PQNB01000001.1 778200:787694 | - | PQNB01000001.1 	778200:782687|


*
| strain | insert | 10kb upstream location | direction of insert | insert location | number of inserts |10KB upstream insert location on 2007 genome | chromosome |
| --- | --- | --- | --- | --- |--- | --- |---|
| 6 (b13) | 1.BSA4upstream | PQNB01000001.1 768685 | + || 1|NC_009042.1 1972553|Chromosome 2|
| 53 (b16)| 1.BSA4upstream | PQNB01000001.1 778195 | - | |1|NC_009042.1 1963057|Chromosome 2|
| 55 (b18)| 1.BSA4upstream | PQNB01000001.1 768685 | + | |1|NC_009042.1 1972553|Chromosome 2|
|88 (b19 )| 1.BSA4upstream | PQNB01000001.1 778195 | - | |1|NC_009042.1 1963058|Chromosome 2|
| | | | |||
| 6 (b13) | 2.GTDNUdownstream| PQNB01000023.1 97903 |+| | *2 (+,-) |NC_009045.1 554770|chromosome 5|
| 53 (b16) | 2.GTDNUdownstream| PQNB01000019.1 593474|-| | *1|NC_009068.1 1580399|chromosome 1|
| 55 (b18) | 2.GTDNUdownstream | PQNB01000022.1 605582 |+||*2 (+,-)|NC_009044.1 881874|chromose 4|
| 88 (b19) | 2.GTDNUdownstream | PQNB01000028.1 440888 |-|| *2 (-,-)|NC_009068.1 3058614|chromosome 1|
