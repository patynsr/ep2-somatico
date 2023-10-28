# ep2-somatico
Trabalho EP2 Somático

# Instalar sratoolskit
```bash
brew install sratoolkit
```

```bash
pip install parallel-fastq-dump
```

```bash
wget -c https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/3.0.0/sratoolkit.3.0.0-ubuntu64.tar.gz
tar -zxvf sratoolkit.3.0.0-ubuntu64.tar.gz
export PATH=$PATH://workspace/somaticoEP2/sratoolkit.3.0.0-ubuntu64/bin/
echo "Aexyo" | sratoolkit.3.0.0-ubuntu64/bin/vdb-config
```

# Download do arquivo WP312
```bash
time parallel-fastq-dump --sra-id SRR8856724 \
--threads 4 \
--outdir ./ \
--split-files \
--gzip
```

# Instalar BWA
```bash
brew install bwa
```

# Instalar samtools
```bash
brew install samtools
```

# Download do arquivo no formato FASTA do genoma humano hg19
```bash
wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr9.fa.gz
```

# BWA index do arquivo chr9.fa.gz
```bash
gunzip chr9.fa.gz
```

```bash
bwa index chr9.fa
```

```bash
samtools faidx chr9.fa
```

# Download das referências do hg19

```bash
wget -c https://storage.googleapis.com/gatk-best-practices/somatic-b37/Mutect2-WGS-panel-b37.vcf
```

```bash
wget -c https://storage.googleapis.com/gatk-best-practices/somatic-b37/Mutect2-WGS-panel-b37.vcf.idx
```

```bash
wget -c  https://storage.googleapis.com/gatk-best-practices/somatic-b37/af-only-gnomad.raw.sites.vcf
```

```bash
wget -c  https://storage.googleapis.com/gatk-best-practices/somatic-b37/af-only-gnomad.raw.sites.vcf.idx
```


# Combinar com pipes: bwa + samtools view e sort

```bash
NOME=WP312; Biblioteca=Nextera; Plataforma=illumina;
bwa mem -t 10 -M -R "@RG\tID:$NOME\tSM:$NOME\tLB:$Biblioteca\tPL:$Plataforma" chr9.fa SRR8856724_1.fastq.gz SRR8856724_2.fastq.gz | samtools view -F4 -Sbu -@2 - | samtools sort -m4G -@2 -o WP312_sorted.bam
```

# Remover duplicata de PCR

```bash
samtools rmdup WP312_sorted.bam WP312_sorted_rmdup.bam
```

```bash
samtools index WP312_sorted_rmdup.bam
```

# Adicionando chr nos VCFs do Gnomad e PoN 

```bash
grep "\#" af-only-gnomad.raw.sites.vcf > af-only-gnomad.raw.sites.chr.vcf
grep  "^9" af-only-gnomad.raw.sites.vcf |  awk '{print("chr"$0)}' >> af-only-gnomad.raw.sites.chr.vcf
```

```bash
grep "\#" Mutect2-WGS-panel-b37.vcf > Mutect2-WGS-panel-b37.chr.vcf 
grep  "^9" Mutect2-WGS-panel-b37.vcf |  awk '{print("chr"$0)}' >> Mutect2-WGS-panel-b37.chr.vcf 
```

## indexando

```bash
bgzip af-only-gnomad.raw.sites.chr.vcf
tabix -p vcf af-only-gnomad.raw.sites.chr.vcf.gz
```

```bash
bgzip Mutect2-WGS-panel-b37.chr.vcf 
tabix -p vcf Mutect2-WGS-panel-b37.chr.vcf.gz
```

# Instalação do bedtools

```bash
brew install bedtools
```

# Gerando BED do arquivo BAM

```bash
bedtools bamtobed -i WP312_sorted_rmdup.bam > WP312_sorted_rmdup.bed
```

```bash
bedtools merge -i WP312_sorted_rmdup.bed > WP312_sorted_rmdup_merged.bed
```

```bash
bedtools sort -i WP312_sorted_rmdup_merged.bed > WP312_sorted_rmdup_merged_sorted.bed
```

# Cobertura Média

```bash
bedtools coverage -a WP312_sorted_rmdup_merged_sorted.bed \
-b WP312_sorted_rmdup.bam -mean \
> WP312_coverageBed.bed
```

# Filtro por total de reads >=20

```bash
cat WP312_coverageBed.bed | \
awk -F "\t" '{if($4>=20){print}}' \
> WP312_coverageBed20x.bed
```

# Instalação GATK

```bash
wget -c https://github.com/broadinstitute/gatk/releases/download/4.2.2.0/gatk-4.2.2.0.zip
```

```bash
unzip gatk-4.2.2.0.zip
```

# Gerar arquivo .dict

```bash
./gatk-4.2.2.0/gatk CreateSequenceDictionary -R chr9.fa -O chr9.dict
```

# Gerar interval_list do chr9

```bash
./gatk-4.2.2.0/gatk ScatterIntervalsByNs -R chr9.fa -O chr9.interval_list -OT ACGT
```

# Converter Bed para Interval_list

```bash
./gatk-4.2.2.0/gatk BedToIntervalList -I WP312_coverageBed20x.bed \
-O WP312_coverageBed20x.interval_list -SD chr9.dict
```

# GATK4 - Mutect Call (Refs hg19 com chr)

```bash
./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I WP312_sorted_rmdup.bam  \
	-V af-only-gnomad.raw.sites.chr.vcf.gz  \
	-L WP312_coverageBed20x.interval_list \
	-O WP312.table
```

```bash
./gatk-4.2.2.0/gatk CalculateContamination \
-I WP312.table \
-O WP312.contamination.table
```

```bash
./gatk-4.2.2.0/gatk Mutect2 \
  -R chr9.fa \
  -I WP312_sorted_rmdup.bam \
  --germline-resource af-only-gnomad.raw.sites.chr.vcf.gz  \
  --panel-of-normals Mutect2-WGS-panel-b37.chr.vcf.gz \
  --disable-sequence-dictionary-validation \
  -L WP312_coverageBed20x.interval_list \
  -O WP312.somatic.pon.vcf.gz
```

```bash
./gatk-4.2.2.0/gatk FilterMutectCalls \
-R chr9.fa \
-V WP312.somatic.pon.vcf.gz \
--contamination-table WP312.contamination.table \
-O WP312.filtered.pon.vcf.gz
```

# Adicionar chr no arquivo para comparação

```bash
zgrep "\#" WP312.filtered.vcf.gz > header.txt
```

```bash
zgrep -v "\#" WP312.filtered.vcf.gz | grep ^9 | awk '{print("chr"$0)}' > variants.txt
```

```bash
cat header.txt variants.txt > WP312.filtered.chr.vcf
```

```bash
bgzip WP312.filtered.chr.vcf
tabix -p vcf WP312.filtered.chr.vcf.gz
```

# Instalação vcftools

```bash
brew install vcftools
```

# Comparação

```bash
vcf-compare WP312.filtered.pon.vcf.gz WP312.filtered.chr.vcf.gz
```
