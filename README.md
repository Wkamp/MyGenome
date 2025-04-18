# Bm88315 Genome Assembly
## Pre-Processing
### 1. Analyzing Sequence Quality
The Bm88315 sequence data was first analyzed using fastqc.
```
fastqc -t 2 Bm88315_1.fq Bm88315_2.fq -o pretrimmed_fastqc_output
```
**Fastqc Output Pages:**
* [Forward](https://wkamp.github.io/MyGenome/data/fastqc_output/pretrimmed_Bm88315_1_fastqc.html)
* [Backward](https://wkamp.github.io/MyGenome/data/fastqc_output/pretrimmed_Bm88315_2_fastqc.html)

The fastqc analysis shows that overall I have a pretty high quality sequence, however there's some adapter contamination and overrepresented sequences which need to be trimmed away.
| ![Per-base sequence quality in the foward sequence](data/fastqc_output/pretrimmed_forward_quality.png) | 
|:--:| 
| *Screenshot of the per-base sequence quality in the foward sequence.* |

| ![Forward adapter contamination](data/fastqc_output/pretimmed_forward_adapter.png) | 
|:--:| 
| *Screenshot of the adapter contamination in the forward sequence.* |

### 2. Trimming the Sequence
The sequence was trimmed using Trimmomatic 0.38.
```
java -jar trimmomatic-0.38.jar PE -threads 2 -phred33 -trimlog Bm_errorlog.txt -summary trim_summary.txt Bm88315_1.fq Bm88315_2.fq Bm88315_1_paired.fq Bm88315_1_unpaired.fq Bm88315_2_paired.fq Bm88315_2_unpaired.fq ILLUMINACLIP:adaptors.fasta:2:30:10 SLIDINGWINDOW:20:20 MINLEN:150
```
**Trimmomatic Summary:**
* Input Read Pairs: 7808561
* Both Surviving Reads: 5981967
* Both Surviving Read Percent: 76.61%
* Forward Only Surviving Reads: 156422
* Forward Only Surviving Read Percent: 2.00%
* Reverse Only Surviving Reads: 1249998
* Reverse Only Surviving Read Percent: 16.01%
* Dropped Reads: 420174
* Dropped Read Percent: 5.38%

### 3. Analyzing Trimmed Sequence
Once again I am using fastqc for analysis.
```
fastqc -t 2 Bm88315_1_paired.fq Bm88315_1_unpaired.fq Bm88315_2_paired.fq Bm88315_2_unpaired.fq -o trimmed_fastqc_output
```
**Fastqc Output Pages:**
* [Forward Paired](https://wkamp.github.io/MyGenome/data/fastqc_output/trimmed_Bm88315_1_paired_fastqc.html)
* [Foward Unpaired](https://wkamp.github.io/MyGenome/data/fastqc_output/trimmed_Bm88315_1_unpaired_fastqc.html)
* [Backward Paired](https://wkamp.github.io/MyGenome/data/fastqc_output/trimmed_Bm88315_2_paired_fastqc.html)
* [Backward Unpaired](https://wkamp.github.io/MyGenome/data/fastqc_output/trimmed_Bm88315_2_unpaired_fastqc.html)

As you can see below, the trimming process managed to almost completely remove all adapter contamination. There is however an anomalous overrepresented sequence of all G's in the reverse read, but it shouldn't pose a problem for our genome assembly. 
| ![Per-base sequence quality in the reverse paired sequence](data/fastqc_output/trimmed_reverse_paired_quality.png) | 
|:--:| 
| *Screenshot of the per-base sequence quality in the reverse paired sequence.* |

| ![adapter contamination in the reverse paired sequence](data/fastqc_output/trimmed_reverse_paired_adapter.png) | 
|:--:| 
| *Screenshot of the adapter contamination in the reverse paired sequence.* |

### 4. Counting Remaining Bases:
I want to know how many bases of the paired data survived the trimming process.
```
awk 'NR%4==2' Bm88315_1_paired.fq | grep -o "[ATCG]" | wc -l
awk 'NR%4==2' Bm88315_2_paired.fq | grep -o "[ATCG]" | wc -l
```
**Base Counts**
* Forward Paired: 897,156,993 
* Reverse Paired: 897,217,181
* Total: 1,794,374,174

## Genome Assembly
### 1. Initial Run
I'm using [Velvet](https://en.wikipedia.org/wiki/Velvet_assembler), more specifically [VelvetOptimiser](https://github.com/tseemann/VelvetOptimiser/tree/master) to assemble the genome. VelvetOptimiser uses Velvet to find the optimal kmer value; you just give it a range and step size.
```
sbatch velvetoptimiser_noclean.sh Bm88315 61 131 10
```
*kmer_start=61, kmer_end=131, step=10*

**Output:**
* Assembly score: 40737045
* Velveth version: 1.2.10
* Velvetg version: 1.2.10
* Readfile(s): -shortPaired -fastq -separate forward.fq reverse.fq
* Velveth parameter string: auto_data_101 101  -shortPaired -fastq -separate forward.fq reverse.fq
* Velvetg parameter string: auto_data_101  -clean no -clean yes -exp_cov 10 -cov_cutoff 2.88539630635769
* Velvet hash value: 101
* Roadmap file size: 667836908
* Total number of contigs: 8171
* n50: 23166
* length of longest contig: 150834
* Total bases in contigs: 42117401
* Number of contigs > 1k: 2924
* Total bases in contigs > 1k: 40737045
* Paired Library insert stats:
* Paired-end library 1 has length: 231, sample standard deviation: 103
* Paired-end library 1 has length: 232, sample standard deviation: 104

The optimal kmer value is the velvet hash value, so 101.

### 2. Optimal Run
In-order to get the most optimized kmer value I want to use a lower step size; the initial run will be used to narrow down the range. I want the previously found optimal kmer of 101 to be the middle of my narrowed down range, and I will half the total range. Previous range length was 70, 70/2 = 35, however 35/2 = 17.5, and I need to start on an odd number, so I will round up to 18. Final range is: [101 - 18, 101 + 18] = [83, 119], step=2.

```
sbatch velvetoptimiser_noclean.sh Bm88315 83 119 2
```

**Output:**
* Assembly score: 40734870
* Velveth version: 1.2.10
* Velvetg version: 1.2.10
* Readfile(s): -shortPaired -fastq -separate forward.fq reverse.fq
* Velveth parameter string: auto_data_97 97  -shortPaired -fastq -separate forward.fq reverse.fq
* Velvetg parameter string: auto_data_97  -clean no -clean yes -exp_cov 11 -cov_cutoff 3.17393593699346
* Velvet hash value: 97
* Roadmap file size: 680108857
* Total number of contigs: 4175
* n50: 29272
* length of longest contig: 151383
* Total bases in contigs: 41374414
* Number of contigs > 1k: 2503
* Total bases in contigs > 1k: 40734870
* Paired Library insert stats:
* Paired-end library 1 has length: 231, sample standard deviation: 103
* Paired-end library 1 has length: 232, sample standard deviation: 104

The most optimal kmer is 97.

## Post-Processing
### 1. Formatting & Culling
Unfortunately there is no standard format for sequence headers, but I'm going to change the format Velvet gives to:
\>Bm88315_contig#, where # is the contig number.
```
perl SimpleFastaHeaders.pl contigs.fa
```
(This script also renames contigs.fa to Bm88315_nh.fasta)

<br>

Additionally, before I run BUSCO in the next step I need to cull the small contigs (length < 200).
```
perl CullShortContigs.pl Bm88315_nh.fasta
```
(This script also renames Bm88315_nh.fasta to Bm88315_final.fasta)

**Output:**
* Contigs Remaing: 3,934
* Genome Size: 41,327,699

### 2. Assembly Quality Analysis
I'm using [BUSCO](https://stab.st-andrews.ac.uk/wiki/index.php/BUSCO) to make sure the genome assembly is of high quality.
```
sbatch BuscoSingularity.sh Bm88315_final.fasta
```
**Output:**
* C:97.6%[S:97.4%, D:0.2%], F:0.8%, M:1.6%, n:1706, E:3.9%	   
* 1665	Complete BUSCOs (C)	(of which 65 contain internal stop codons)		   
* 1662	Complete and single-copy BUSCOs (S)	   
* 3	Complete and duplicated BUSCOs (D)	   
* 14	Fragmented BUSCOs (F)			   
* 27	Missing BUSCOs (M)			   
* 1706	Total BUSCO groups searched		   

Assembly Statistics:
* 3934	Number of scaffolds
* 6950	Number of contigs
* 41327699	Total length
* 0.221%	Percent gaps
* 29 KB	Scaffold N50
* 13 KB	Contigs N50


Dependencies and versions:
* hmmsearch: 3.1
* bbtools: 39.06
* miniprot_index: 0.13-r248
* miniprot_align: 0.13-r248
* python: sys.version_info(major=3, minor=7, micro=12, releaselevel='final', serial=0)
* busco: 5.7.0

## Blast
### 1. MoMitochondrion
```
singularity run --app blast2120 /share/singularity/images/ccs/conda/amd-conda1-centos8.sinf blastn -query MoMitochondrion.fasta -subject Bm88315_final.fasta -evalue 1e-50 -max_target_seqs 20000 -outfmt '6 qseqid sseqid slen length qstart qend sstart send btop' -out MoMitochondrion.Bm88315_final.BLAST
```

#### Checking Output Size
The BLAST file should have about 40kb contig size.
```
awk '{print $2, $3}' MoMitochondrion.Bm88315_final.BLAST | sort -u | awk '{sum += $2} END {print sum}'
```
**Output**: 93,633

This is 93kb, which is way too large.
Peering into the BLAST file you can see there's an issue with contig967:
| ![Bm88315 MoMitochondrion Blast file](data/Bm88315_blast_screenshot.png) | 
|:--:| 
| *Screenshot of the Bm88315 MoMitochondrion Blast file.* |

Contig967 itself is 48kb; exactly why is unclear, it's possible there was a false join. Regardless, to fix it I'm going to trim from position 46498 to 48509.
```
awk -v start=46498 -v end=48509 '
BEGIN {keep_len = end - start + 1}
/^>/ {
    if (seq != "") {
        if (header ~ />Bm88315_contig967/) {
            seq = substr(seq, 1, start - 1) substr(seq, end + 1)
        }
        print header
        print seq
    }
    header = $0
    seq = ""
    next
}
{
    seq = $0
}
END {
    if (header ~ />Bm88315_contig967/) {
        seq = substr(seq, 1, start - 1) substr(seq, end + 1)
    }
    print header
    print seq
}' Bm88315_final.fasta > trimmed_Bm88315_final.fasta
```
### 2. MoMitochondrion Again
```
singularity run --app blast2120 /share/singularity/images/ccs/conda/amd-conda1-centos8.sinf blastn -query MoMitochondrion.fasta -subject trimmed_Bm88315_final.fasta -evalue 1e-50 -max_target_seqs 20000 -outfmt '6 qseqid sseqid slen length qstart qend sstart send btop' -out MoMitochondrion.trimmed_Bm88315_final.BLAST
```
#### Checking Output Size Again
```
awk '{print $2, $3}' MoMitochondrion.Bm88315_final.BLAST | sort -u | awk '{sum += $2} END {print sum}'
```
**Output**: 45,124

45kb is an acceptable size.

### 2. B71v2sh
```
singularity run --app blast2120 /share/singularity/images/ccs/conda/amd-conda1-centos8.sinf blastn -query B71v2sh_masked.fasta -subject trimmed_Bm88315_final.fasta -evalue 1e-50 -max_target_seqs 20000 -outfmt '6 qseqid sseqid slen length qstart qend sstart send btop' -out B71v2sh.trimmed_Bm88315_final.BLAST
```

#### Contig List for NCBI
```
awk '$3/$4 > 0.9 {print $2 ",mitochondrion"}' B71v2sh.trimmed_Bm88315_final.BLAST > Bm88315_mitochondrion.csv
```

## Genome Prediction
