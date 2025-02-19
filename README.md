# MyGenome

## 1. Analyzing sequence quality
The Bm88315 sequence data was analyzed using fastqc
```
fastqc -t 2 Bm88315_1.fq Bm88315_2.fq -o pretrimmed_fastqc_output
```
Fastqc Output:
* [Forward](https://wkamp.github.io/MyGenome/data/fastqc_output/pretrimmed_Bm88315_1_fastqc.html)
* [Backward](https://wkamp.github.io/MyGenome/data/fastqc_output/pretrimmed_Bm88315_2_fastqc.html)

## 2. Trimming the sequence
The sequence was trimmed using Trimmomatic 0.38
```
java -jar trimmomatic-0.38.jar PE -threads 2 -phred33 -trimlog Bm_errorlog.txt Bm88315_1.fq Bm88315_2.fq Bm88315_1_paired.fq Bm88315_1_unpaired.fq Bm88315_2_paired.fq Bm88315_2_unpaired.fq ILLUMINACLIP:adaptors.fasta:2:30:10 SLIDINGWINDOW:20:20 MINLEN:150
```
## 3. Analyzing trimmed sequence
```
fastqc -t 2 Bm88315_1_paired.fq Bm88315_1_unpaired.fq Bm88315_2_paired.fq Bm88315_2_unpaired.fq -o trimmed_fastqc_output
```
