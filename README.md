*TEPID*
======

TEPID uses illumina sequencing reads to identify novel TE variants. First, reads are mapped to a reference genome using bowtie2 and yaha. This produces two bam files, one containing mapped reads, and one containing mapped split reads. Python scripts are then used to identify the absence of reference TE insertions and the presence of non-reference insertions. Having a high-quality TE annotation and reference genome assembly, as well as deep sequencing coverage (>20x) and long reads (100 bp or more) will greatly improve the quality of TE presence/absence calls made.

## Installation

You need miniconda to install the requirements.

```bash
git clone https://github.com/rimjhimroy/TEPID-fix.git
cd ./TEPID
conda env create --file env/tepid.yaml
conda activate tepid
python setup.py install
```
## Fix TEPID

By: Rimjhim Roy Choudhury  
Here TEPID is Version: 0.8  
Tepid-map has been modified at the sorting step with to adapt to the samtools version:
`samtools sort -@ $proc -o "${fname}.bam" "${fname}_unsort.bam"`  
Tepid-discover has been modified in calc_cov method to accomodate genomes with no chromosomes but only scaffolds  
Tepid-refine has been modified to fix error related to global variable "options" not being found in the function `process_missed`.  

## Testing

This will check that the code runs correctly and is able to identify a known, experimentally verified, TE insertion and TE deletion.

```
python setup.py test
```

## Usage

There are two modes that TEPID can be run in, single-end or paired-end mode.  

### Step 1: Mapping

#### Paired-end mode

Use the `tepid-map` script to map paired-end reads.

```bash
tepid-map -- map paired-end data using bowtie2 and yaha
  -h  show help and exit
  -x  path to bowtie2 index
  -y  path to yaha index
  -p  number of cores to use
  -s  average insert size
  -n  sample name
  -1  fastq file with #1 mates
  -2  fastq file with #2 mates
  -r  recursive (optional)
  -z  gzip fastq files (optional)
```

This will look for the two fastq file specified using the `-1` and `-2` options, and map these using the number of processors specified in `-p`. These files must be present in the current directory, or in direcories immediately below the current directory if the `-r` option is used.

#### Single-end mode

Use the `tepid-map-se` script to map single-end reads.  

```bash
tepid-map-se -- map single-end data using bowtie2 and yaha
  -h  show help and exit
  -x  path to bowtie2 index
  -y  path to yaha index
  -p  number of cores to use
  -n  sample name
  -q  fastq file containing reads
  -r  recursive (optional)
  -z  gzip fastq files (optional)
```


This will give you the following files:

* [name].bam
* [name].bam.bai
* [name].split.bam
* [name].umap.fastq (this will be compressed if you selected the `-z` option)

The name of these output files will come from the option given with the `-n` flag.

Next, go to the directory containing your bam files and run the `tepid-discover` script to identify TE variants.

### Step 2: TE variant discovery

```bash
usage: tepid-discover [-h] [--version] [-k] [-d | -i] [--strict] [--mask MASK]
                      [-D DISCORDANT] [-p PROC] -n NAME -c CONC -s SPLIT -t TE
                      [--se]

TEPID -- transposable element polymorphism identification

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  -k, --keep            keep all intermediate files
  -d, --deletions       find deletions only
  -i, --insertions      find insertions only
  --strict              Report high-confidence variants only
  --mask MASK           Mask chromosomes in comma separated list or file
  -D DISCORDANT, --discordant DISCORDANT
                        Supply discordant reads bam file
  -p PROC, --proc PROC  number of processors
  -n NAME, --name NAME  sample name
  -c CONC, --conc CONC  bam file from bowtie2
  -s SPLIT, --split SPLIT
                        split reads bam file from yaha
  -t TE, --te TE        TE annotation bedfile
  --se                  Run in single-end mode
```

The following TE annotations for use with TEPID are included in the repository:  
  - *Arabidopsis thaliana* (TAIR9 and TAIR10)
  - *Brachypodium distachyon*
  - *Homo sapiens* (hg19)

If using a different TE annotation, the file _must_ have tab-separated columns in the format:

`chromosome start stop strand TE_name TE_family TE_superfamily`

Output files:

  * TE insertions bedfile
  * TE deletions bedfile (TE present in reference but not sample)
  * File containing names of reads providing evidence for insertions
  * File containing names of reads providing evidence for deletions

### Step 3: Refinement and genotyping

```bash
usage: tepid-refine [-h] [--version] [-k] [-i INSERTIONS] [-d DELETIONS]
                    [-p PROC] -t TE -n NAME -c CONC -s SPLIT -a ALL_SAMPLES

TEPID -- refine TE insertion and deletion calls

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  -k, --keep            keep all intermediate files
  -i INSERTIONS, --insertions INSERTIONS
                        File containing collapsed TE insertions for all
                        samples in population
  -d DELETIONS, --deletions DELETIONS
                        File containing collapsed TE deletions for all samples
                        in population
  -p PROC, --proc PROC  number of processors
  -t TE, --te TE        TE annotation bedfile
  -n NAME, --name NAME  sample name
  -c CONC, --conc CONC  bam file from bowtie2
  -s SPLIT, --split SPLIT
                        split reads bam file from yaha
  -a ALL_SAMPLES, --all_samples ALL_SAMPLES
                        List of all sample names
```

This step is optional and for groups of related samples only, such as a population or generational study. First, a file containing all idenetified variants for the group needs to be generated, with a list of samples that contain each insertion as a column in a bedfile:

```bash
[chromosome name]	[start position]	[stop position]	[TE name] [comma-separated list of samples containing insertion]
```

For example:
```bash
chr1	23094	23200	AT1TE69285	Sorbo,Nok-3
```

To do this, the `merge_insertions.py` and `merge_deletions.py` scripts included in the TEpy package, in the `Scripts/` directory, can be used. A list of all sample names is also needed (one sample name on each line of a file).


## Required Tools

#### Command-line tools

* [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) >= v2.1.0  
* [yaha](https://github.com/GregoryFaust/yaha) >= v0.1.82  
* [samtools](http://www.htslib.org/download/) > v1.3  
* [samblaster](https://github.com/GregoryFaust/samblaster) >= v0.1.19 (needed for paired-end data only)  
* [bedtools](http://bedtools.readthedocs.org/en/latest/) >= v2.25.0  

#### Python requirements

* [python](https://www.python.org) v2.7  
* [numpy](http://www.numpy.org/)  
* [pybedtools](http://pythonhosted.org/pybedtools/)  
* [pysam](http://pysam.readthedocs.org/en/latest/)  


## Citation

If you use this software if your work, please cite:

Stuart T, Eichten SR, Cahn J, Karpievitch Y, Borevitz JO, Lister R. Population scale mapping of transposable element diversity reveals links to gene regulation and epigenomic variation. eLife. 2016. [doi:10.7554/eLife.20777](http://dx.doi.org/10.7554/eLife.20777)
