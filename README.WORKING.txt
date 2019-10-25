#TEPID is Version: 0.8
#Tepid-map has been modified at the sorting step with to adapt to the samtools version:
samtools sort -@ $proc -o "${fname}.bam" "${fname}_unsort.bam"
#Tepid-discover has been modified in calc_cov method to accomodate genomes with scaffolds only
#This installation of TEPID works with the following version of programs
Bowtie2 version 2.3.4.3
Samblaster: Version 0.1.24
YAHA version 0.1.83
Samtools Version: 1.7
bedtools v2.25.0
python 2.7.12
numpy==1.11.0
pybedtools==0.7.1
pysam==0.8.4



