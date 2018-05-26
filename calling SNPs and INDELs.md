# Calling SNP 位点和INDEL
## 使用GATK
### 1 使用GATK将BAM文件转换为GVCF文件
```更新到GATK4后此步骤省略
```
### 2 将GVCF文件转换为VCF文件
```
gatk CombineGVCFs -R reference.fa -V Sample1.gvcf.gz -V Sample2.gvcf.gz -V ... -O Pop.vcf.gz
```
### 使用GATKcall SNP并进行条件过滤
#### 首先进行Variant select
使用默认参数
##### SNP
```
gatk SelectVariants -R /reference.fa -V Pop.vcf.gz -select-type SNP -O Pop.SNP.vcf.gz
```
##### INDEL
```
gatk SelectVariants -R reference.fa -V Pop.vcf.gz -select-type INDEL -O Pop.INDEL.vcf.gz
```
#### 对SNP和INDEL质量进行条件过滤（Hard Filter）
```
gatk VariantFiltration -R reference.fa -V Pop.SNP.vcf.gz -G-filter "QD < 2.0 || FS > 60.0 || MQ < 40.0 || MQRankSum < -12.5 || ReadPosRankSum < -8.0"  -G-filter-name "my_snp_filter" -O Pop.HDflt.SNP.vcf.gz
```
```
gatk VariantFiltration -R reference.fa -V Pop.INDEL.vcf.gz -G-filter "QD < 2.0 || FS > 200.0 || ReadPosRankSum < -20.0" -G-filter-name "my_indel_filter" -O Pop.HDflt.INDEL.vcf.gz
```
