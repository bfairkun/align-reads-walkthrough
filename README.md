Here are instructions to download raw data from Xu et al (Nature Methodsvolume 16, pages55–58 (2019)) (SRR7208764\_[1|2].fastq.gz), and align the data. This is an MPE-seq library in wildtype-yeast, sequenced to depth ~5M paired end (60bp + 15bp) reads on a NextSeq platform.

First, log onto midway, and download this repository (which contains some files you will need) with git clone command:
```
git clone https://github.com/bfairkun/align-reads-walkthrough.git
```

Next, download the raw data (sequencing read [.fastq] files) from Xu et al using sra toolkit's fastq-dump command.

```
#load sra-toolkit from RCC available modules
module load sra_toolkit

#download data
fastq-dump --split-files SRR7208764
# should create 2 .fastq files

# take a look at the files:

cat SRR7208764_1.fastq
# can kill a command with CTRL-C

# head is a useful command for looking at the top of large files
head -20 SRR7208764_1.fastq

# gzip compress them both for efficient storage of large files
# * is a wildcard. So *.fastq is expanded to match anything ending in .fastq
gzip ./*.fastq

# can still peek into compressed files by first decompressing the file with 'zcat' then piping that output into the next command with |
zcat SRR7208764_1.fastq.gz | head -20
```

Now we should align reads to a reference sequence, like the yeast reference genome. I use STAR aligner software for aligning reads that potentially have splice sites. I don't think STAR is available from RCC available software modules, so you need to download and install it yourself on Midway. Easiest way to do that is to use conda or miniconda to manage installation of software.

First, download [miniconda](https://docs.conda.io/en/latest/miniconda.html) and install it.
then install STAR using miniconda:
```
conda install -c bioconda star
```
and check that it works by calling STAR with help flag. I recommend browsing the onling documentation for STAR aligner, or at least the 'general usage' section at the beginning.
```
STAR -h
```

To use STAR to align the fastq.gz files, first you need to create a indexed reference genome that the aligner can efficiently use to align reads.

Included in this directory is the yeast reference genome. browse around to find it, take a look at the files. Notice the file extensions, as [.fa], [.gff] are commonly used filetypes. They are all human readable (so you can take a look at the contents of the file with `head` or `cat` or `less`.

Follow the [STAR manual](http://labshare.cshl.edu/shares/gingeraslab/www-data/dobin/STAR/Releases/FromGitHub/STAR-2.6.1a/doc/STARmanual.pdf) online to generate a genome index. You should end up executing a command something like this:

```
#make a new directory to store genome index files
mkdir MyGenome

# use start to create genome index files, (you will have to change a lot of these $values for your files

STAR --runMode genomeGenerate --genomeDir $OutDir/STAR_GenomeDirectory --genomeFastaFiles $genomefasta --sjdbGTFfile $genomegtf --sjdbOverhang 100 --sjdbGTFtagExonParentTranscript Parent --sjdbGTFfeatureExon CDS
```

Finally, align reads. You could use defualt settings at first. For this dataset, I used settings like this for publication (you will have to change a lot of these parameter values to match your files) If you want to know what or why I chose all these alignment parameters, browse the STAR manual and or ask me about it.

```
STAR --genomeDir $OutDir/STAR_GenomeDirectory --readFilesIn $OutDir/temp_files/MyR1s.filtered.fastq $OutDir/temp_files/MyR2s.filtered.fastq --alignIntronMin 20 --alignIntronMax 1100 --outSAMtype BAM SortedByCoordinate --outFileNamePrefix $OutDir/STAR_DuplicateReadsRemovedAlignments/$samplename/ --alignEndsType EndToEnd --clip3pAdapterSeq CTGTCTCTTATACACATCTCCGAGCCCACGAGAC --readMapNumber -1 --clip5pNbases 7 0 --alignMatesGapMax 400 --alignSplicedMateMapLmin 16 --outSAMattributes All --runThreadN 4 --alignSJDBoverhangMin 1 --outSAMmultNmax 1 --outSAMunmapped Within KeepPairs --outFilterMismatchNmax 3
```

This should create alignment files ([.bam] or [.sam]). [.sam] is human readable. browse around, get a sense of what some of the fields are and how the data is stored. bam is the compressed version of [.sam] file. You need to install samtools (use conda to install) to decompress it. For example:

```
samtools view MyBam.bam | head -20
```

Now that you should have aligned reads, it is important to visualize them to make sure the laignment process worked as expected, and the libraries look expected (in the case of MPE-seq, you should check some of the targeted genes and make sure you get a lot of read depth there, and not so much at other genes. There are of course more systematic ways to do this than a genome broswer, but genome browsers are great to give you some intuition).
You can view [.bam] files on a genome browser, but first you need to index it for memory efficient browsing:

```
samtools index MyBam.bam
```

Then, download IGV. I personally use IGV on my personal computer, and either download the bam file to my personal computer or load it in IGV by connecting to Midway through (File>>LoadFromURL)File>>LoadFromURL. I think it is also useful to load a regions file (check out TargetIntrons.sorted.bed file in this repository) to help you find regions of interest in the genome more efficiently.

