## Assessment of the [Drosophila melanogaster ISO1 ref. strain](https://www.ncbi.nlm.nih.gov/biosample/SAMN08511563) genome (69X NanoPore data) assemblies using NextDenovo, Canu, Flye, Shasta and Wtdbg
* **Download reads**  
`SRA Accession: SRR6702603, SRR6821890`

* **Prepare input file (input.fofn)**  
`ls SRR6702603.fasta.gz SRR6821890.fasta.gz > input.fofn`

* **Calculate the recommended minimum seed length**  
`bin/seq_stat -g 130m input.fofn > input.fofn.stat`

The following is the partial content of file input.fofn.stat, and it shows the recommended minimum seed length is **10000** bp at the last line.
```
[Read length stat]
Types            Count (#) Length (bp)
N10                  28670   26040
N20                  67370   20911
N30                 114553   17369
N40                 171113   14487
N50                 239221   11980
N60                 322288    9717
N70                 426190    7645
N80                 561853    5680
N90                 754633    3736

Types               Count (#)           Bases (bp)  Depth (X)
Raw                   1145050           8965907711      68.97
Filtered                    0                    0       0.00
Clean                 1145050           8965907711      68.97

*Suggested seed_cutoff (genome size: 130.00Mb, expected seed depth: 45, real seed depth: 40.48): 10000 bp
```

* **Prepare config file (run.cfg)** 
``` 
[General]
job_type = sge
job_prefix = nextDenovo
task = all 
rewrite = yes 
deltmp = no
rerun = 3
parallel_jobs = 12
input_type = raw
input_fofn = input.fofn
workdir = 01_rundir

[correct_option]
read_cutoff = 1k
seed_cutoff = 10k
blocksize = 2g
pa_correction = 6
seed_cutfiles = 6
sort_options = -m 30g -t 35 -k 45
minimap2_options_raw = -x ava-ont -t 20
correction_options = -p 35

[assemble_option]
minimap2_options_cns = -x ava-ont -t 20 -k17 -w17
nextgraph_options = -a 1
```

* **Run**   
`nohup nextDenovo run.cfg &`

* **Get result**
1. Final corrected reads file (use the '-b' parameter to get more corrected reads):
`01_rundir/02.cns_align/01.seed_cns.sh.work/seed_cns*/cns.fasta`
2. Final assembly result:  
`01_rundir/03.ctg_graph/nd.asm.fasta`

The folowing is the assembly statistics:
```
Type           Length (bp)            Count (#)
N10             25701192                   1
N20             22251987                   2
N30             22251987                   2
N40             21195733                   3
N50             21195733                   3
N60             18110856                   4
N70             13648743                   5
N80              6408543                   6
N90              1033518                  12

Min.               18454                   -
Max.            25701192                   -
Ave.             1826448                   -
Total          133330776                  73
```

* **Assemble with shasta**  
`shasta-Linux-0.5.1  --input SRR6702603.fasta --input SRR6821890.fasta --threads 30` 

* **Download reference**   
```
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/215/GCF_000001215.4_Release_6_plus_ISO1_MT/GCF_000001215.4_Release_6_plus_ISO1_MT_genomic.fna.gz
gzip -d GCF_000001215.4_Release_6_plus_ISO1_MT_genomic.fna.gz
```

* **Run Quast v5.0.2**   
`quast.py --large --eukaryote --min-identity 80 --threads 30 -r GCF_000001215.4_Release_6_plus_ISO1_MT_genomic.fna nextDenovo.asm.fa Canu.asm.fa Flye.asm.fa Shasta.asm.fa Wtdbg.asm.fa`

<a name="quast"></a>  

* **Quast result**

| | nextDenovo | Canu | Flye | Shasta | Wtdbg |
| --------- | ------ | ------ | ------ | ------ | ------ |
| # contigs | 73 | 424 | 461 | 872 | 510 |
| Largest contig | 25701192 | 14715425 | 12613153 | 1801407 | 23221757 |
| Total length | 133330776 | 140540470 | 135880693 | 129225244 | 132926651 |
| N50 | 21195733 | 4298595 | 6016667 | 535885 | 12028162 |
| **NG50** | 18110856 | 4298595 | 6016667 | 440773 | 10631323 |
| N75 | 13648743 | 777595 | 2182645 | 244480 | 3308195 |
| **NG75** | 3925274 | 714013 | 1367004 | 182722 | 1752322 |
| LG50 | 4 | 11 | 9 | 92 | 5 |
| LG75 | 7 | 36 | 20 | 218 | 13 |
| # **misassemblies** | 345 | 971 | 724 | 262 | 616 |
| # misassembled contigs | 48 | 226 | 217 | 78 | 191 |
| # **local misassemblies** | 137 | 433 | 670 | 123 | 185 |
| # unaligned mis. contigs | 1 | 3 | 5 | 7 | 36 |
| # unaligned contigs | 1 + 36 part | 8 + 122 part | 11 + 118 part | 191 + 76 part | 89 + 291 part |
| Unaligned length | 603053 | 769264 | 811595 | 1660668 | 2264882 |
| Genome fraction (%) | 92.109 | 93.614 | 91.799 | 88.085 | 91.504 |
| Duplication ratio | 1.011 | 1.047 | 1.032 | 1.016 | 1.002 |
| # **mismatches per 100 kbp** | 90.86 | 183.12 | 220.48 | 609.69 | 179.86 |
| # **indels per 100 kbp** | 567.78 | 831.54 | 1334.52 | 1428.10 | 1081.15 |
| Largest alignment | 25696021 | 11699048 | 11981267 | 1799773 | 18844039 |
| Total aligned length | 132416893 | 139189216 | 134650393 | 127438699 | 130313270 |
| NA50 | 6618721 | 3863099 | 5596752 | 527231 | 4309906 |
| **NGA50** | 6618721 | 3863099 | 5143715 | 434179 | 4174617 |
| NA75 | 3269191 | 670044 | 1955654 | 230034 | 1573933 |
| **NGA75** | 2125978 | 611559 | 1267543 | 168924 | 928918 |
| LGA50 | 5 | 13 | 11 | 94 | 10 |
| LGA75 | 14 | 42 | 24 | 227 | 27 |

***Note:*** the results of Canu, Flye and Wtdbg are copied from ftp://ftp.dfci.harvard.edu/pub/hli/wtdbg/dm-ISO1, published by [wtdbg2 paper](https://www.nature.com/articles/s41592-019-0669-3), the complete result of Quast can be seen from [here](./TEST4.pdf).