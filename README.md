

### This code is associated with the paper from McCrone et al., "Stochastic processes constrain the within and between host evolution of influenza virus". eLife, 2018. http://dx.doi.org/10.7554/eLife.35962


# Lauring Variant Pipeline

Nominates candidate variants by comparing the sequences in a test sample to those found in a plasmid control.
The Pipeline runs as one phase which takes in fastq files and outputs putative variants as well as all base call above a set frequency.  It is then up to the user to filter the putative variants based on the characteristics provided.

## Directory list
* bin
	* variantPipeline.py : align reads, sort, call variants using deepSNV, and characterize putative variants.
	* variant_pipeline.pbs : an example pbs script used to implement the Pipeline
* doc
	* workflow diagram, examples
* lib
	* supporting libraries (mostly R libraries)
* scripts
	* supporting scripts (bpipe, python, R)
	* note that variantPipeline.bpipe.config holds several default config variables
* test
	* automated tests (mostly python)

* tutorial
	* The directories and instructions needed to run the tutorial. Instructions can be found in tutorial.html.

## bin/variantPipeline.py
 This script is a thin python wrapper around a bpipe pipeline which in turn calls fastqc, pydmx-al, bowtie, picard. Whenever this is launched, the bpipe scripts are overwrittem from the scripts directory and stored in the output directory as a log of what was run.

##UPDATE THESE HERE ##

Usage: variantPipeline.py -i {input_dir} -o {output_dir} -r {reference} -p {plasmid control name} -a {p.val cutoff} -m {fisher,max,average} -d {one.sided,two.sided}


* Inputs:  
	* -i dir containing left fastq, right fastq named as sample.read_direction[1,2].#.fastq
		* the python scripts change_names* can be used as a fast means of renaming fastq to this format.
	* -r path to the reference genome for alignment made using

	```bash
	bowtie2-build WSN33.fa WSN33
	```
	Where WSN33.fa is your fasta file
	* -p plasmid control : the sample name of the plasmid control fastq.
	* -a alpha : The p value cut off to used in the final output. Any variant with a p.val>a will be removed
	* -m method : The method used to combine the p value from each strand "fisher","average","max".
	* -t test :Boolean switch to run program in test mode. Everything will be set up but bpipe will run in test mode
	* -d Dispersion : Dispersion estimation to be used in deepSNV. Options are c("two.sided","one.sided","bin"). Anything other 	than two.sided or one.sided will yield a binomial distribution. "two.sided" and "one.sided" estimate dispersion and use a beta binomial to model nucleotide counts. "two.sided" uses a two sided test and is most conservative and appropriate when dealing with high number of  PCR cycles.
	* -bam bam : Boolean switch. Sometimes it is useful to rerun the deepSNV variant calling with different parameters (e.g. dispersion, p.combine). Activating this  takes bam files as inputs so you don\'t have to rerun the alignment and sorting. In this case the input directory should contain the bam files.



	See the tutorial for more information.

	*NOTE: Your fasta is used in the variant calling step and needs to end in .fa*


* Outputs:
	* __01_fastqc__ : zips containing a brief summary report on quality of input fastqs
	* __03_align__ : bam files from bowtie2 alignment
	* __04_remove_duplicates__ : bam files with duplicates marked by picard
	* __Variants__ : sum.csv files made by adding mapping quality, read position, and phred information to the putative variants found in the variant output, and reads.csv containing similar information for each read responsible for a variant
	* __deepSNV__ : csv files of the deepSNV variant calls (bonferroni p<0.1), coverage data for each sample (*.cov.csv) .fa consensus sequences
		* an empty csv is made for the plasmid control in order to appease bpipe.
		* the variant csvs in this directory does not include data concerning read position, mapQ, or Phred information.  That can be foun in the Variants dir.
	* Variants and deepSNV contain files begining in "all" these files contain the output for each sample concatenated into one file for your viewing and analyzing pleasure.
	* __doc__ : Bpipe makes a cool/qausi-useful html report in doc/index.html
	* __.bpipe__: a hidden directory that contains information bpipe uses to restart failed runs.  *Note: It is sometimes useful to delete this and start fresh*


## Dependencies

Note : *Flux is the name of the computing core used by our lab at the Univeristy of Michigan. Some of the directions may be specific to those working on this platform*

The pipeline comes with many of the required programs (bpipe and pycard); however, bowtie2, samtools and certain R  and python libraries are required by the variant calling.

Note: *The R package deepSNV may need to be installed under your username on Flux.  The other dependencies are simply added as modules.*

To open R on flux simply type
```
module load med
module load R/3.1.1
R
```


This can be done from any directory.
* deepSNV

```
source("http://bioconductor.org/biocLite.R")
biocLite("deepSNV")
```

You will be prompted to install in a local directory beginning with ~/. This means you are installing in your home directory and the library will be available just to you.  Installation should take a while.

Adapted and developed by JT McCrone based on work done by
Chris Gates/Peter Ulintz
UM BCRCF Bioinformatics Core
