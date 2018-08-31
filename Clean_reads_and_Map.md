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

「#### 获得INDEL的区间--此步忽略
```
java -jar GenomeAnalysisTK.jar -nt 30 -Rref.fasta -T RealignerTargetCreator -o 3.realign/sample.realn.intervals -I 2.rehead/sample.rmdup.bam 2>3.realign/sample.realn.intervals.log
```
#### 对这些区间重新比对--此步忽略
```
java -jar GenomeAnalysisTK.jar -R ref.fa -T IndelRealigner -targetIntervals 3.realign/sample.realn.intervals -o 3.realign/sample.realn.bam -I 2.rehead/sample.rmdup.bam 2>3.realign/sample.realn.bam.log
```
##gvcf 文件转换为vcf文件，并把所有个体合在一起

java -jar /home/software/GenomeAnalysisTK.jar -T GenotypeGVCFs -R ../../reference_stacei/Bstacei_316_v1.0.softmasked.fa  -V AS2_13.vcf.gz -V AS2_14.vcf.gz -V AS2_15.vcf.gz -V AS2_18.vcf.gz -V AS40.vcf.gz -V AS4.vcf.gz -V ES_15.vcf.gz -V ES_1.vcf.gz -V ES_3.vcf.gz -V ES6_12.vcf.gz -V ES6_17.vcf.gz -V ES6_13.vcf.gz -V ES6_1.vcf.gz -V ES6_4.vcf.gz -V ES6_5.vcf.gz -V ES_6.vcf.gz -V ES_9.vcf.gz -o Brachy.vcf.gz

##提取SNP文件命令行
/usr/bin/java -jar /home/software/GenomeAnalysisTK.jar -T SelectVariants -R ../../reference_stacei/Bstacei_316_v1.0.softmasked.fa -V Brachy.vcf.gz -selectType SNP -o Brachy.SNP.vcf.gz

##提取INDEL命令行
/usr/bin/java -jar /home/software/GenomeAnalysisTK.jar -T SelectVariants -R ../../reference_stacei/Bstacei_316_v1.0.softmasked.fa -V Pop.vcf.gz -selectType INDEL -o Pop.INDEL.vcf.gz

##过滤SNP命令行
/usr/bin/java -jar /home/software/GenomeAnalysisTK.jar -T VariantFiltration -R ../../reference_stacei/Bstacei_316_v1.0.softmasked.fa -V Brachy.SNP.vcf.gz --filterExpression "QD < 2.0 || FS > 60.0 || MQ < 40.0 || MQRankSum < -12.5 || ReadPosRankSum < -8.0"  --filterName "my_snp_filter" -o Brachy.HDflt.SNP.vcf.gz

###过滤INDEL命令行
gatk VariantFiltration -R ../../reference_stacei/Bstacei_316_v1.0.softmasked.fa -V Brachy.INDEL.vcf.gz -G-filter "QD < 2.0 || FS > 200.0 || ReadPosRankSum < -20.0" -G-filter-name "my_indel_filter" -O Brachy.HDflt.INDEL.vcf.gz



##检测HWE 使用命令vcftools --gzvcf 01.perl.filter/01.Barchy.SNP.vcf.gz --keep AS.sample.list --hardy --out 01.ASHWEBrachy.out


###将bash改为zsh
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zs
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
source ~/.zsh

##HWE 命令行

vcftools --gzvcf 01.perl.filter/01.Barchy.fix.SNP.vcf.gz --keep 01.perl.filter/ES.sample.list --hardy --out 01.ESHWEBrachy.out &

## 把哈登温伯格平衡检验不合格的位点去除，第一个是标记的不合格位点，第二个为输入文件，第三那个为输出文件
perl 00.change.sample.pl 01.Barchy.SNP.vcf.gz 01.Barchy.fix.SNP.vcf.gz
