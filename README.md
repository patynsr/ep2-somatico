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
brew install sratoolkit
```time parallel-fastq-dump --sra-id SRR8856724 \
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


