# Vic-TrAnScRiPtOmIcS
A SUMMARY OF CODES FOR TRANSCRIPTOME ASSEMBLY

## 0. REQUIREMENTS
```r
## consideraciones ##
## TRINITY fue instalado a traves de CONDA ##
## TRINITY requiere de "samtools 1.3" ##
## descarga "samtools 1.3" de https://github.com/trinityrnaseq/trinityrnaseq/wiki/Running-Trinity ##
## ingresa a la pestana "files" ##
## encontraras archivos con nombre "linux-64/samtools-1.3.1-h60f3df9_12.tar.bz2" ##
## el enlace es https://anaconda.org/bioconda/samtools/1.3.1/download/linux-64/samtools-1.3.1-h60f3df9_12.tar.bz2 ##
## TRINITY requiere que todo el contenido de "bin" (sobre todo "samtools") se encuentre en "/home/hp/anaconda3/envs/trinity/bin"
## corrobora que "samtools" dentro de esta carpeta es la version 1.3 ##
```

## 1. download FASTQ from SRA ##
```r
prefetch --max-size 50G --option-file list.txt ;
mv */*.sra . ;
# fasterq-dump --split-files *.sra ; #
fastq-dump --defline-seq '@$sn[_$rn]/$ri' --split-files *.sra 
gzip -t 12 *fastq ;
fastqc * ;
ls ;
```

## 2. run TRINITY ##
```r
mkdir fasta/ fasta.gene_trans_map/ timing/ ;
for s1 in *.gz
do
s2=$(basename $s1 .fastq.gz)
Trinity --seqType fq --max_memory 12G --single $s1 --CPU 4 --output ${s2}.trinity.out ;
mv ${s2}.trinity.out/Trinity.fasta ${s2}.trinity.out/${s2}.Trinity.fasta ;
mv ${s2}.trinity.out/Trinity.fasta.gene_trans_map ${s2}.trinity.out/${s2}.Trinity.fasta.gene_trans_map ;
mv ${s2}.trinity.out/Trinity.timing ${s2}.trinity.out/${s2}.Trinity.timing ;
mv ${s2}.trinity.out/${s2}.Trinity.fasta fasta/ ;
mv ${s2}.trinity.out/${s2}.Trinity.fasta.gene_trans_map fasta.gene_trans_map/ ;
mv ${s2}.trinity.out/${s2}.Trinity.timing timing/;
done
ls ;
```
