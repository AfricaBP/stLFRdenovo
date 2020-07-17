# stLFRdenovo
**StLFRdenovo** is a De Novo assembly pipeline to deal with barcoded reads.  It is based on Supernova, with a fastq parsing and sorting module constumized for stLFR data.

## Availability
The pre-compiled version of StLFRdenovo can be downloaded from here: https://github.com/BGI-biotools/stLFRdenovo/releases
## Structure of the files
```
├── bin
│   ├── BcAssembly
│   ├── BcScaffold
│   ├── FillGaps
│   ├── MakeFasta
│   ├── ParseBarcodedFastqs
│   └── PeAssembly
├── script
│   ├── barcode_list.txt
│   ├── SOAPfilter_v2.2
│   ├── split_barcode.pl
│   └── split_barcode.sh
├── step_1_prepare_data.sh
├── step_2_run_assembly.sh
└── stLFRdenovo_pipeline.sh
```
## System Requirement
It only runs on linux system like Centos or Ubuntu. In order to run stLFRdenovo on large genomes like human, the computing node should have at least 512 GB and 1TB free disk space (The final output would not take that much space, it is just for intermediate files generated by the software).

## Useage
+ 1st, create a new work folder
```
> mkdir YourProjectName
> cd YourProjectName
```
+ 2nd, run the 1-step executable script or run it by 2 steps.
    + 2.1 run the 1-step executable script with raw reads.
    ```
    > stLFRdenovo_dir/stLFRdenovo_pipeline.sh [ -f raw_fastq_1 raw_fastq_2 ] [ -o out_path ] [ -m max_memory = 512 ] [ -t thread_num = 32 ] 
    ```
    This script will do data pre-processing step (generating two separate read pair file in fastq format) and the assembly step automatically.
    + 2.2 run the pipeline by 2 steps.
        + step 1, data pre-processing
        Before running assembly, the raw fastq file's barcode should be separated, and the duplicate and adapter also need to be filtered. if you have clean fastq file, you can juamp to 2nd.
        ```
        > stLFRdenovo_dir/step_1_prepare_data.sh [ -f raw_fastq_1 raw_fastq_2 ] [ -o output_prefix ] [ -t thread_num = 32 ]
        ```
        + step 2, running assembly
        After prepare data, you can start assembly with two clean fastq files. Then execute `step_2_run_assembly.sh` without any parameter to see how to use it.
        ```
        > stLFRdenovo_dir/step_2_run_assembly.sh [ -f clean_fastq_1 clean_fastq_2 ] [ -o out_path ] [ -m max_memory = 512 ] [ -t thread_num = 32 ] 
        ```
        A typical way to launch the pipeline is:
        ```
        > stLFRdenovo_dir/step_2_run_assembly.sh -f read1.fq[.gz] read2.fq[.gz] -t 32 -m 512 -o out_path
        ```
        If `out_path` doesn't exist, the pipeline will create it.
        Please Note that the maximum threads to use in stLFRdenovo is 32, if you specify more than 32, the pipeline would use 32 threads at most. Also, the memory limit is just a reminder here how much memory you want to use for now, it would not quit the pipeline even when it uses more memory than what you have specified.

After the pipeline finishes, there will be a `scaffolds.fasta.gz` file in the `out_path`, which is the final assembly result.
If you have any questions, please contact liuchao3@genomics.cn.


## <a name=appendix>Appendix</a>

### <a name=stlfr_raw>Appendix A : an example of stLFR raw reads</a>

- Read1 of stLFR raw reads

  *There is no barcode information within read1*

```
@CL100117953L1C001R001_1/1
GGCCGGGGACTCAGGGAGAGATTCTAAGCTCTCTGTATGTGGAAGGCTCGCGAGACAGGAAACACACAAGACACGGGCGTTGTATACAGGTTCGGGCCGC
+
FGFFFFEF9FEFFFFFCF;FEA4EFE4FFFFGFGFCFFFEEFDDFE??@FCFCF<FGFEDFFF=F@)FFFF:D;9FCFF(:GEECG8F?F>E/FFCFFF0
@CL100117953L1C001R001_3/1
CCACCACGCCATTTGCTCCGAGAAGACAGCTACAACGCATGCGTATATAATAGAAACGCTCGTGCAAAACAAACTATATATAAAAAAATGATGACCAATG
+
<BE3BBAECB@CEBFBBD2F/EA>ABABDECFEEFCCAEB@BE%>8AAEAECE/;B=E6D<F6EC@FD93DB7E@;=??E/FFDDEB;EF;EF%C4E?EE
@CL100117953L1C001R001_4/1
CACCGATCCACAAGAGGCCTTACAGAATGGGAGCCAGCGAGTTGGCGGAAGTCAAGAAGCAAGTCGATGAACAGTTATAGAAGGGATACATCCGTCCGAG
+
FFFFFFFFFFFFFFEF>FFFFFFFFFFEFFFFFFFFFFFFFFEFFFFFCEF>FF>GCAFBFFFFFF;7FEFFCFE9FAFFEFFFF<=;'8EFFFAF=FFF
```

- Read2 of stLFR raw reads

  *The last 42-bp sequence provides the stLFR barcode information within read2*

  *There are 3 acceptable types of stLFR raw read2: 126bp or 142bp or 154bp. If your read2 does not belong to one of them, then it will stop.*

```
@CL100117953L1C001R001_1/2
GGGTATTACGCTTCTCAGCGGCCCGAACCTGTATACATCGCCCGTGTCTTGTGTGTTTCCTGTCTCGCGAGCCTTCCACATACAGAGAGCTTAGAATCTCATTACTAACGTCTTNTCTACAATACGAGGTTTTTGAGGAGAC
+
FFF*BF5FFFFGFFEFFFFEFFFFG@AFFFFFFFFFFFFFFFEFFFEFFFFEFDFBFDEFFGAFFFFFF(FEFFFFFEFD<AFFF<FB7FFFFF>;3EEEFFFFCFFFFFCE@6!0FFE;FFFFFF6.*D,CAFFFFFEFFF
@CL100117953L1C001R001_3/2
CAACAGCACTAGCTGGTGTCACGTGATCATCTCTGNAGATTTTCCTCTCTCTTCGCCTGGCGATGGGTTAGCACATTGNCAGAGGTAGTATCTATCANCGCAGGTATAAGCACTNCTCGAACGCCAATATTAAAGTTCGACC
+
.14,.<84,9B9(<?8>B,381B'?+841'9/69<!46;8'+?;3.,81693)48/1@/+)819&,6+344,69;(48!98%?6/+7/%))1.,(>.!))3<>91@@DBC%C&+!.;4C.44@4;E4'%'>9)1@AE3B:3?
@CL100117953L1C001R001_4/2
AGTCAACGCACATCCTCTTGGTTTTGTCTTTCTTCTCCACAAAGATAACCGGAGCACCCCAAGGCGACGTGCTCGGACGGATGTATCCCTTCTATAACGGCGACCACTGATCTGAGGCGTTAACGCGATGATTTGACATCTC
+
EFD8DBFFEFFFFEFFFFFFFFDCDFEFEFFFFFFFFFFFCEFFEEFFFFFFDFFFFFEFFEFFFFDFFDFFFFFF>EFFD>F<6FFFD'FF9FA;,D+9FFBFFFFFFD&>5(&>FFFFDFEFFFFE(F5FDFFEFFFFFF
```

### <a name=stlfr>Appendix B : an example of stLFR reads</a>

- Read1 of stLFR reads

  *The barcode information is appended at the header line. #xxx_xxx_xxx part is the barcode.*

```
@CL100117953L1C001R001_1#844_1383_927/1 1       1
GGCCGGGGACTCAGGGAGAGATTCTAAGCTCTCTGTATGTGGAAGGCTCGCGAGACAGGAAACACACAAGACACGGGCGTTGTATACAGGTTCGGGCCGC
+
FGFFFFEF9FEFFFFFCF;FEA4EFE4FFFFGFGFCFFFEEFDDFE??@FCFCF<FGFEDFFF=F@)FFFF:D;9FCFF(:GEECG8F?F>E/FFCFFF0
@CL100117953L1C001R001_3#1469_851_342/1 2       1
CCACCACGCCATTTGCTCCGAGAAGACAGCTACAACGCATGCGTATATAATAGAAACGCTCGTGCAAAACAAACTATATATAAAAAAATGATGACCAATG
+
<BE3BBAECB@CEBFBBD2F/EA>ABABDECFEEFCCAEB@BE%>8AAEAECE/;B=E6D<F6EC@FD93DB7E@;=??E/FFDDEB;EF;EF%C4E?EE
@CL100117953L1C001R001_4#643_1473_309/1 3       1
CACCGATCCACAAGAGGCCTTACAGAATGGGAGCCAGCGAGTTGGCGGAAGTCAAGAAGCAAGTCGATGAACAGTTATAGAAGGGATACATCCGTCCGAG
+
FFFFFFFFFFFFFFEF>FFFFFFFFFFEFFFFFFFFFFFFFFEFFFFFCEF>FF>GCAFBFFFFFF;7FEFFCFE9FAFFEFFFF<=;'8EFFFAF=FFF
```

- Read2 of stLFR reads

  *The barcode information is appended at the header line. #xxx_xxx_xxx part is the barcode.*

```
@CL100117953L1C001R001_1#844_1383_927/2 1       1
GGGTATTACGCTTCTCAGCGGCCCGAACCTGTATACATCGCCCGTGTCTTGTGTGTTTCCTGTCTCGCGAGCCTTCCACATACAGAGAGCTTAGAATCTC
+
FFF*BF5FFFFGFFEFFFFEFFFFG@AFFFFFFFFFFFFFFFEFFFEFFFFEFDFBFDEFFGAFFFFFF(FEFFFFFEFD<AFFF<FB7FFFFF>;3EEE
@CL100117953L1C001R001_3#1469_851_342/2 2       1
CAACAGCACTAGCTGGTGTCACGTGATCATCTCTGNAGATTTTCCTCTCTCTTCGCCTGGCGATGGGTTAGCACATTGNCAGAGGTAGTATCTATCANCG
+
.14,.<84,9B9(<?8>B,381B'?+841'9/69<!46;8'+?;3.,81693)48/1@/+)819&,6+344,69;(48!98%?6/+7/%))1.,(>.!))
@CL100117953L1C001R001_4#643_1473_309/2 3       1
AGTCAACGCACATCCTCTTGGTTTTGTCTTTCTTCTCCACAAAGATAACCGGAGCACCCCAAGGCGACGTGCTCGGACGGATGTATCCCTTCTATAACGG
+
EFD8DBFFEFFFFEFFFFFFFFDCDFEFEFFFFFFFFFFFCEFFEEFFFFFFDFFFFFEFFEFFFFDFFDFFFFFF>EFFD>F<6FFFD'FF9FA;,D+9
```
