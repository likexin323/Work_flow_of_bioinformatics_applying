# Calling SNP 位点和INDEL
## 使用GATK
### 1 使用GATK将BAM文件转换为GVCF文件
```
/usr/bin/java -jar /home/share/users/yangyongzhi2012/tools/GATK/GenomeAnalysisTK.jar -T HaplotypeCaller -R reference.fa  -I sample.realn.bam -nct 30 -ERC GVCF -o sample.gvcf.gz -variant_index_type LINEAR -variant_index_parameter 128000 2>&1 | Sample.gvcf.log
```
### 2 将GVCF文件转换为VCF文件
```
/usr/bin/java -jar /home/share/users/yangyongzhi2012/tools/GATK/GenomeAnalysisTK.jar -T GenotypeGVCFs -R reference.fa -V Sample1.gvcf.gz -V Sample2.gvcf.gz -V ... -o Pop.vcf.gz
```
### 使用GATKcall SNP并进行条件过滤
#### 首先进行Variant select
使用默认参数
##### SNP
```
/usr/bin/java -jar /home/share/users/yangyongzhi2012/tools/GATK/GenomeAnalysisTK.jar -T SelectVariants -R /reference.fa -V Pop.vcf.gz -selectType SNP -o Pop.SNP.vcf.gz
```
##### INDEL
```
/usr/bin/java -jar /home/share/users/yangyongzhi2012/tools/GATK/GenomeAnalysisTK.jar -T SelectVariants -R reference.fa -V Pop.vcf.gz -selectType INDEL -o Pop.INDEL.vcf.gz
```
#### 对SNP和INDEL质量进行条件过滤（Hard Filter）
```
/usr/bin/java -jar /home/share/users/yangyongzhi2012/tools/GATK/GenomeAnalysisTK.jar -T VariantFiltration -R reference.fa -V Pop.SNP.vcf.gz --filterExpression "QD < 2.0 || FS > 60.0 || MQ < 40.0 || MQRankSum < -12.5 || ReadPosRankSum < -8.0"  --filterName "my_snp_filter" -o Pop.HDflt.SNP.vcf.gz
```
```
/usr/bin/java -jar /home/share/users/yangyongzhi2012/tools/GATK/GenomeAnalysisTK.jar -T VariantFiltration -R reference.fa -V Pop.INDEL.vcf.gz --filterExpression "QD < 2.0 || FS > 200.0 || ReadPosRankSum < -20.0" --filterName "my_indel_filter" -o Pop.HDflt.INDEL.vcf.gz
```