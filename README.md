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
## TRINITY requiere que todo el contenido de "bin" (sobre todo "samtools") se encuentre en "/home/hp/anaconda3/envs/trinity/bin" ##
## corrobora que "samtools" dentro de esta carpeta es la version 1.3 ##
## puede detectarse algun error con "salmon" ##
## instalar "tbb" en el environment de Trinity ##
## conda install -c bioconda tbb=2020.2 ##

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

## 2. Trinity requiere que los reads presenten la denominación @1/1 y @1/2, donde el número a la izquiera de "/" denota el número de read en el fastq-file y el número a la derecha de "/" debe ser "1" para forward y "2" para reverse, para ello se puede emplear el siguiente comando ##
```r
por ejemplo, si los 3 primeros reads del fastq forward "SRR7003713_f_paired.fq.gz" tienen los siguientes headers

@SRR7003713.5 5 length=125
@SRR7003713.8 8 length=125
@SRR7003713.12 12 length=125

despues de correr el comando 

for r1 in *_f_paired.fq.gz
do
prefix=$(basename $r1 _f_paired.fq.gz)
r2=${prefix}_r_paired.fq.gz
zcat $r1 | sed -e "s/^@${prefix}./@/g" | sed -e 's/\ .*/\/1/g' > ${prefix}_f_paired2.fq ;
zcat $r2 | sed -e "s/^@${prefix}./@/g" | sed -e 's/\ .*/\/2/g' > ${prefix}_r_paired2.fq ;
done ; 
gzip *2.fq ;
ls -lh *.gz ;

se obtendran los nuevos headers
@5/1
@8/1
@12/1

y los nuevos FASTQ-FILES tendrán la extensión "paired2.fq.gz" que serán procesados por "Trinity"
```

## 3. run TRINITY ##
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

mkdir fasta/ fasta.gene_trans_map/ timing/ ;
for r1 in *_f_paired2.fq.gz
do
prefix=$(basename $r1 _f_paired2.fq.gz)
r2=${prefix}_r_paired2.fq.gz
Trinity --seqType fq --max_memory 28G --right $r1 --left $r2 --CPU 14 --output ${prefix}.trinity.out ;
mv ${prefix}.trinity.out/Trinity.fasta ${prefix}.trinity.out/${prefix}.Trinity.fasta ;
mv ${prefix}.trinity.out/Trinity.fasta.gene_trans_map ${prefix}.trinity.out/${prefix}.Trinity.fasta.gene_trans_map ;
mv ${prefix}.trinity.out/Trinity.timing ${prefix}.trinity.out/${prefix}.Trinity.timing ;
mv ${prefix}.trinity.out/${prefix}.Trinity.fasta fasta/ ;
mv ${prefix}.trinity.out/${prefix}.Trinity.fasta.gene_trans_map fasta.gene_trans_map/ ;
mv ${prefix}.trinity.out/${prefix}.Trinity.timing timing/;
done ;
ls ;
```

## 3.1. compress very large FASTQ files ##
```r
# 3.1.1 #
for t1 in *.sra
do
prefix=$(basename $t1 .sra)
./fastq-dump --defline-seq '@$sn[_$rn]/$ri' --split-files $t1 
rm $t1 ; 
done 
ls ; 

# 3.1.2 #

for r1 in *_1.fastq
do
r2=$(basename $r1 _1.fastq)
echo "vic, starting with sample $r2"

echo "vic, don't be ass, I'm compressing file ${r2}_1.fastq"
cat ${r2}_1.fastq | parallel --pipe --recend '' gzip -9 > ${r2}_1.fastq.gz ;
chmod 777 ${r2}_1.fastq.gz ;
rm ${r2}_1.fastq ; 
echo "vic, sample ${r2}_1.fastq was removed"

echo "vic, don't be ass, I'm compressing file ${r2}_2.fastq" 
cat ${r2}_2.fastq | parallel --pipe --recend '' gzip -9 > ${r2}_2.fastq.gz ;
chmod 777 ${r2}_2.fastq.gz ;
rm ${r2}_2.fastq ; 
echo "vic, sample ${r2}_2.fastq was removed"

echo "vic, done FORWARD and REVERSE for sample ${r2}, moving to next sample"
echo "..."
done 

echo "no more samples, all files were sucessfully compressed"
ls ;
```
