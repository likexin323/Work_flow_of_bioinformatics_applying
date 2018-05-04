# clean_reads  and map
## Clean Reads
一般使用脚本 01.PairEndMultyReadsQualityFilter.pl
```
perl 01.PairEndMultyReadsQualityFilter.pl $prefix $fastq1 $fastq2
```
## 将reads比对到参考基因组
### 为参考基因组建立索引
```
bwa index ref.fasta
samtools faidx ref.fasta
java -jar /home/share/software/picard/picard-tools-1.129/picard.jar CreateSequenceDictionary REFERENCE=ref.fasta OUTPUT=ref.fasta
```
### 使用BWA软件进行map
#### 初次比对：使用BWA软件MEM模块，默认参数
```
bwa mem -t 30 -R '@RG\tID:$samplename\tPL:illumina\tPU:illumina\tLB:$samplename\tSM:$samplename\t' ref.fasta sample.1.fq.gz sample.2.fq.gz | samtools sort -O bam -T /tmp/$samplename -o outdir/outprefix
```
#### 合并有多段序列的个体样品bam文件
```
samtools merge sample.sort.bam sample.L1.bam sample.L2.bam ...
```
#### 去除重复序列
```
java -Xmx10g -jar /home/share/software/picard/picard-tools-1.129/picard.jar MarkDuplicates INPUT=1.bwa/sample.sort.bam OUTPUT=2.rehead/sample.rmdup.bam METRICS_FILE=2.rehead/sample.dup.txt REMOVE_DUPLICATES=true ; samtools index 2.rehead/sample.rmdup.bam
```
#### 获得INDEL的区间
```
java -jar GenomeAnalysisTK.jar -nt 30 -Rref.fasta -T RealignerTargetCreator -o 3.realign/sample.realn.intervals -I 2.rehead/sample.rmdup.bam 2>3.realign/sample.realn.intervals.log
```
#### 对这些区间重新比对
```
java -jar GenomeAnalysisTK.jar -R ref.fa -T IndelRealigner -targetIntervals 3.realign/sample.realn.intervals -o 3.realign/sample.realn.bam -I 2.rehead/sample.rmdup.bam 2>3.realign/sample.realn.bam.log
```
