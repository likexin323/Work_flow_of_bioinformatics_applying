# 01.Clean_reads  and map
## Clean Reads
一般使用脚本 01.PairEndMultyReadsQualityFilter.pl
```
perl 01.PairEndMultyReadsQualityFilter.pl $prefix $fastq1 $fastq2
```
####### 也可以采用fastp软件进行过滤，该软件可以同时把低质量的、较短的、以及含有接头的序列同时过滤掉，而且不需要输入接头序列

.....

fastp -l 45 -q 20 -i /data/users/likexin/brachypodium/raw/ES6_13/ES6_13_HGWVWALXX_L2_1.fq.gz -I /data/users/likexin/brachypodium/raw/ES6_13/ES6_13_HGWVWALXX_L2_2.fq.gz -o /data/users/likexin/brachypodium/clean/ES6_13_1.fq.gz -O /data/users/likexin/brachypodium/clean/ES6_13_2.fq.gz

......

## 将reads比对到参考基因组
### 为参考基因组建立索引
```
bwa index ref.fasta
samtools faidx ref.fasta
java -jar /home/share/software/picard/picard-tools-1.129/picard.jar CreateSequenceDictionary REFERENCE=ref.fasta OUTPUT=ref.fasta
```
### 使用BWA软件进行map
#### 初次比对：使用BWA软件MEM模块，默认参数

此处修改样本名称的时候需要把美元符号（$)去掉，如果一个文库测多次，需要分开做，然后使用samtools merge将得到的bam文件合并

这里同一个文库多次测得的数据，在单引号内的名称需要一致，其他地方可以使用a,b等区分开。
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
生成vcf文件

gatk --java-options "-Xmx4g" HaplotypeCaller -R /home/lzu_zhangshangzhe/00.genome/GCF_000622305.1_S.galili_v1.0_genomic.fa -L Scaffold17170  -I /home/lzu_zhangshangzhe/00.calling.SNP/Cgr-1.rmdup.bam -ERC GVCF -O /home/lzu_zhangshangzhe/00.calling.SNP/03.realn.bam/Cgr-1.Scaffold17170.g.vcf.gz

#########GATK4到此就可以了，不再做下面两步

#### 获得INDEL的区间
```
java -jar GenomeAnalysisTK.jar -nt 30 -Rref.fasta -T RealignerTargetCreator -o 3.realign/sample.realn.intervals -I 2.rehead/sample.rmdup.bam 2>3.realign/sample.realn.intervals.log
```
#### 对这些区间重新比对
```
java -jar GenomeAnalysisTK.jar -R ref.fa -T IndelRealigner -targetIntervals 3.realign/sample.realn.intervals -o 3.realign/sample.realn.bam -I 2.rehead/sample.rmdup.bam 2>3.realign/sample.realn.bam.log
```
