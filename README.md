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

# Remover duplicata de PCR ****RODAR****

```bash
samtools rmdup WP312_sorted.bam WP312_sorted_rmdup.bam
```

# Adicionando chr nos VCFs do Gnomad e PoN ****RODAR****

```bash
grep "\#" af-only-gnomad.raw.sites.vcf > af-only-gnomad.raw.sites.chr.vcf
```

```bash
grep  "^9" af-only-gnomad.raw.sites.vcf |  awk '{print("chr"$0)}' >> af-only-gnomad.raw.sites.chr.vcf
```
