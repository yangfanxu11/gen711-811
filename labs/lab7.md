---
title: "Lab 7"
author: "Jeffrey Miller"
date: "2026 Mar 6th"
---

# Today: Writing Scripts and Working with Data
- Discuss group projects
- Transfer plots and results to/from RON
- Downloading files directly to RON



## Instructions for Group projects

See an example of the final github product of the final projects [here](https://github.com/jthmiller/gen711-811-example)

Instructions for starting group projects can be found [here](https://github.com/jthmiller/gen711-811-example/blob/main/Group-Project-Git_instructions.md)




```bash
wget ftp://ftp.ensemblgenomes.org/pub/release-37/bacteria/species_EnsemblBacteria.txt
```

or

```bash
$ cd
$ curl -O ftp://ftp.ensemblgenomes.org/pub/release-37/bacteria/species_EnsemblBacteria.txt
```

# Part A

::::::::::::::::::::::::::::::::::::::: objectives

- Why study *E. coli*?
- Understand the data set.
- What is hypermutability?

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- What data are we using?
- Why is this experiment important?

::::::::::::::::::::::::::::::::::::::::::::::::::

## Background

We are going to use a long-term sequencing dataset from a population of *Escherichia coli*.

- **What is *E. coli*?**
  - *E. coli* are rod-shaped bacteria that can survive under a wide variety of conditions including variable temperatures, nutrient availability, and oxygen levels. Most strains are harmless, but some are associated with food-poisoning.

![](fig/172px-EscherichiaColi_NIAID.jpg){alt='Wikimedia'}

<!-- https://species.wikimedia.org/wiki/Escherichia_coli#/media/File:EscherichiaColi_NIAID.jpg -->

- **Why is *E. coli* important?**
  - *E. coli* are one of the most well-studied model organisms in science. As a single-celled organism, *E. coli* reproduces rapidly, typically doubling its population every 20 minutes, which means it can be manipulated easily in experiments. In addition, most naturally occurring strains of *E. coli* are harmless. Most importantly, the genetics of *E. coli* are fairly well understood and can be manipulated to study adaptation and evolution.

## The data

- The data we are going to use is part of a long-term evolution experiment led by [Richard Lenski](https://en.wikipedia.org/wiki/E._coli_long-term_evolution_experiment).

- The experiment was designed to assess adaptation in *E. coli*. A population was propagated for more than 40,000 generations in a glucose-limited minimal medium (in most conditions glucose is the best carbon source for *E. coli*, providing faster growth than other sugars). This medium was supplemented with citrate, which *E. coli* cannot metabolize in the aerobic conditions of the experiment. Sequencing of the populations at regular time points revealed that spontaneous citrate-using variant (**Cit+**) appeared between 31,000 and 31,500 generations, causing an increase in population size and diversity. In addition, this experiment showed hypermutability in certain regions. Hypermutability is important and can help accelerate adaptation to novel environments, but also can be selected against in well-adapted populations.

- To see a timeline of the experiment to date, check out this [figure](https://en.wikipedia.org/wiki/E._coli_long-term_evolution_experiment#/media/File:LTEE_Timeline_as_of_May_28,_2016.png), and this paper [Blount et al. 2008: Historical contingency and the evolution of a key innovation in an experimental population of *Escherichia coli*](https://www.pnas.org/content/105/23/7899).

### View the metadata

We will be working with three sample events from the **Ara-3** strain of this experiment, one from 5,000 generations, one from 15,000 generations, and one from 50,000 generations. The population changed substantially during the course of the experiment, and we will be exploring how (the evolution of a **Cit+** mutant and **hypermutability**) with our variant calling workflow. The metadata file associated with this lesson can be [downloaded directly here](files/Ecoli_metadata_composite.csv) or [viewed in Github](https://github.com/datacarpentry/wrangling-genomics/blob/main/episodes/files/Ecoli_metadata_composite.csv). If you would like to know details of how the file was created, you can look at [some notes and sources here](https://github.com/datacarpentry/wrangling-genomics/blob/main/episodes/files/Ecoli_metadata_composite_README.md).

This metadata describes information on the *Ara-3* clones and the columns represent:

| Column           | Description                                     | 
| ---------------- | ----------------------------------------------- |
| strain           | strain name                                     | 
| generation       | generation when sample frozen                   | 
| clade            | based on parsimony-based tree                   | 
| reference        | study the samples were originally sequenced for | 
| population       | ancestral population group                      | 
| mutator          | hypermutability mutant status                   | 
| facility         | facility samples were sequenced at              | 
| run              | Sequence read archive sample ID                 | 
| read\_type        | library type of reads                           | 
| read\_length      | length of reads in sample                       | 
| sequencing\_depth | depth of sequencing                             | 
| cit              | citrate-using mutant status                     | 

:::::::::::::::::::::::::::::::::::::::  challenge

### Challenge

Based on the metadata, can you answer the following questions?

1. How many different generations exist in the data?

cut -d, -f2 Ecoli_metadata_composite.csv | sort | uniq

2. How many rows and how many columns are in this data?

wc -l Ecoli_metadata_composite.csv
head -n1 Ecoli_metadata_composite.csv | tr ',' '\n' | wc -l

3. How many citrate+ mutants have been recorded in **Ara-3**?

cut -d, -f12 Ecoli_metadata_composite.csv | tail -n +2 | grep plus | wc -l

4. How many hypermutable mutants have been recorded in **Ara-3**?


cut -d, -f6 Ecoli_metadata_composite.csv | tail -n +2 | grep plus | wc -l


::::::::::::::::::::::::::::::::::::::: objectives

- Explain how a FASTQ file encodes per-base quality scores.
- Interpret a FastQC plot summarizing per-base quality across all reads.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How can I describe the quality of my data?

::::::::::::::::::::::::::::::::::::::::::::::::::

## Bioinformatic workflows

When working with high-throughput sequencing data, the raw reads you get off of the sequencer will need to pass
through a number of  different tools in order to generate your final desired output. The execution of this set of
tools in a specified order is commonly referred to as a *workflow* or a *pipeline*.

An example of the workflow we will be using for our variant calling analysis is provided below with a brief
description of each step.

![](fig/variant_calling_workflow.png){alt='workflow'}

1. Quality control - Assessing quality using FastQC
2. Quality control - Trimming and/or filtering reads (if necessary)
3. Align reads to reference genome
4. Perform post-alignment clean-up
5. Variant calling

```bash
mkdir -p ~/gen711-811/data/untrimmed_fastq/
cd ~/gen711-811/data/untrimmed_fastq
```
Download the files (do not use unless instructed.See faster option)
```
curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/004/SRR2589044/SRR2589044_1.fastq.gz
curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/004/SRR2589044/SRR2589044_2.fastq.gz
curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/003/SRR2584863/SRR2584863_1.fastq.gz
curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/003/SRR2584863/SRR2584863_2.fastq.gz
curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/006/SRR2584866/SRR2584866_1.fastq.gz
curl -O ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/006/SRR2584866/SRR2584866_2.fastq.gz
```
### Faster option
If your workshop is short on time or the venue's internet connection is weak or unstable, learners can
avoid needing to download the data and instead use the data files provided in the '/tmp/Gen711-811_data/' directory. Run fastqc on all of these files

For each input FASTQ file, FastQC has created a `.zip` file and a

`.html` file. The `.zip` file extension indicates that this is
actually a compressed set of multiple output files. We will be working
with these output files soon. The `.html` file is a stable webpage
displaying the summary report for each of our samples.

We want to keep our data files and our results files separate, so we
will move these
output files into a new directory within our `results/` directory.

```bash
$ mkdir -p ~/gen711-811/results/fastqc_untrimmed_reads
$ mv *.zip ~/gen711-811/results/fastqc_untrimmed_reads/
$ mv *.html ~/gen711-811/results/fastqc_untrimmed_reads/
```

Download the html files to view them.


# Part B

::::::::::::::::::::::::::::::::::::::: objectives

- Clean FASTQ reads using Trimmomatic.
- Select and set multiple options for command-line bioinformatic tools.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How can I get rid of sequence data that does not meet my quality standards?

::::::::::::::::::::::::::::::::::::::::::::::::::

## Cleaning reads

In the previous episode, we took a high-level look at the quality
of each of our samples using FastQC. We visualized per-base quality
graphs showing the distribution of read quality at each base across
all reads in a sample and extracted information about which samples
fail which quality checks. Some of our samples failed quite a few quality metrics used by FastQC. This does not mean,
though, that our samples should be thrown out! It is very common to have some quality metrics fail, and this may or may not be a problem for your downstream application. For our variant calling workflow, we will be removing some of the low quality sequences to reduce our false positive rate due to sequencing error.

We will use a program called
[Trimmomatic](https://www.usadellab.org/cms/?page=trimmomatic) to
filter poor quality reads and trim poor quality bases from our samples.

## Trimmomatic options

Trimmomatic has a variety of options to trim your reads. If we run the following command, we can see some of our options.

```bash
$ trimmomatic
```

Which will give you the following output:

```output
Usage: 
       PE [-version] [-threads <threads>] [-phred33|-phred64] [-trimlog <trimLogFile>] [-summary <statsSummaryFile>] [-quiet] [-validatePairs] [-basein <inputBase> | <inputFile1> <inputFile2>] [-baseout <outputBase> | <outputFile1P> <outputFile1U> <outputFile2P> <outputFile2U>] <trimmer1>...
   or: 
       SE [-version] [-threads <threads>] [-phred33|-phred64] [-trimlog <trimLogFile>] [-summary <statsSummaryFile>] [-quiet] <inputFile> <outputFile> <trimmer1>...
   or: 
       -version
```

This output shows us that we must first specify whether we have paired end (`PE`) or single end (`SE`) reads.
Next, we specify what flag we would like to run. For example, you can specify `threads` to indicate the number of
processors on your computer that you want Trimmomatic to use. In most cases using multiple threads (processors) can help to run the trimming faster. These flags are not necessary, but they can give you more control over the command. The flags are followed by positional arguments, meaning the order in which you specify them is important.
In paired end mode, Trimmomatic expects the two input files, and then the names of the output files. These files are described below. While, in single end mode, Trimmomatic will expect 1 file as input, after which you can enter the optional settings and lastly the name of the output file.

| option         | meaning                                                                                                      | 
| -------------- | ------------------------------------------------------------------------------------------------------------ |
| \<inputFile1>   | Input reads to be trimmed. Typically the file name will contain an `_1` or `_R1` in the name.                                          | 
| \<inputFile2>   | Input reads to be trimmed. Typically the file name will contain an `_2` or `_R2` in the name.                                          | 
| \<outputFile1P> | Output file that contains surviving pairs from the `_1` file.                                                          | 
| \<outputFile1U> | Output file that contains orphaned reads from the `_1` file.                                                           | 
| \<outputFile2P> | Output file that contains surviving pairs from the `_2` file.                                                          | 
| \<outputFile2U> | Output file that contains orphaned reads from the `_2` file.                                                           | 

The last thing trimmomatic expects to see is the trimming parameters:

| step           | meaning                                                                                                      | 
| -------------- | ------------------------------------------------------------------------------------------------------------ |
| `ILLUMINACLIP`               | Perform adapter removal.                                                                                     | 
| `SLIDINGWINDOW`               | Perform sliding window trimming, cutting once the average quality within the window falls below a threshold. | 
| `LEADING`               | Cut bases off the start of a read, if below a threshold quality.                                             | 
| `TRAILING`               | Cut bases off the end of a read, if below a threshold quality.                                               | 
| `CROP`               | Cut the read to a specified length.                                                                          | 
| `HEADCROP`               | Cut the specified number of bases from the start of the read.                                                | 
| `MINLEN`               | Drop an entire read if it is below a specified length.                                                       | 
| `TOPHRED33`               | Convert quality scores to Phred-33.                                                                          | 
| `TOPHRED64`               | Convert quality scores to Phred-64.                                                                          | 

We will use only a few of these options and trimming steps in our
analysis. It is important to understand the steps you are using to
clean your data. For more information about the Trimmomatic arguments
and options, see [the Trimmomatic manual](https://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf).

However, a complete command for Trimmomatic will look something like the command below. This command is an example and will not work, as we do not have the files it refers to:

```bash
$ trimmomatic PE -threads 4 SRR_1056_1.fastq SRR_1056_2.fastq  \
              SRR_1056_1.trimmed.fastq SRR_1056_1un.trimmed.fastq \
              SRR_1056_2.trimmed.fastq SRR_1056_2un.trimmed.fastq \
              ILLUMINACLIP:SRR_adapters.fa SLIDINGWINDOW:4:20
```

In this example, we have told Trimmomatic:

| code           | meaning                                                                                                      | 
| -------------- | ------------------------------------------------------------------------------------------------------------ |
| `PE`               | that it will be taking a paired end file as input                                                            | 
| `-threads 4`               | to use four computing threads to run (this will speed up our run)                                            | 
| `SRR_1056_1.fastq`               | the first input file name                                                                                    | 
| `SRR_1056_2.fastq`               | the second input file name                                                                                   | 
| `SRR_1056_1.trimmed.fastq`               | the output file for surviving pairs from the `_1` file                                                                | 
| `SRR_1056_1un.trimmed.fastq`               | the output file for orphaned reads from the `_1` file                                                                 | 
| `SRR_1056_2.trimmed.fastq`               | the output file for surviving pairs from the `_2` file                                                                | 
| `SRR_1056_2un.trimmed.fastq`               | the output file for orphaned reads from the `_2` file                                                                 | 
| `ILLUMINACLIP:SRR_adapters.fa`               | to clip the Illumina adapters from the input file using the adapter sequences listed in `SRR_adapters.fa`                     | 
| `SLIDINGWINDOW:4:20`               | to use a sliding window of size 4 that will remove bases if their phred score is below 20                    | 

:::::::::::::::::::::::::::::::::::::::::  callout

## Multi-line commands

Some of the commands we ran in this lesson are long! When typing a long
command into your terminal, you can use the `\` character
to separate code chunks onto separate lines. This can make your code more readable.


::::::::::::::::::::::::::::::::::::::::::::::::::

## Running Trimmomatic

Now we will run Trimmomatic on our data. To begin, navigate to your `untrimmed_fastq` data directory:

```bash
$ cd ~/gen711-811/data/untrimmed_fastq
```

We are going to run Trimmomatic on one of our paired-end samples.
While using FastQC we saw that Nextera adapters were present in our samples.
The adapter sequences came with the installation of trimmomatic, so we will first copy these sequences into our current directory.

### NOTE: This next step will proably not work. Think about using the 'which' command and see if you can find out where these adaptors are

```bash
$ cp ~/.miniconda3/pkgs/trimmomatic-0.38-0/share/trimmomatic-0.38-0/adapters/NexteraPE-PE.fa .
```

We will also use a sliding window of size 4 that will remove bases if their
phred score is below 20 (like in our example above). We will also
discard any reads that do not have at least 25 bases remaining after
this trimming step. Three additional pieces of code are also added to the end
of the ILLUMINACLIP step. These three additional numbers (2:40:15) tell
Trimmimatic how to handle sequence matches to the Nextera adapters. A detailed
explanation of how they work is advanced for this particular lesson. For now we
will use these numbers as a default and recognize they are needed to for Trimmomatic
to run properly. This command will take a few minutes to run.

```bash
$ trimmomatic PE SRR2589044_1.fastq.gz SRR2589044_2.fastq.gz \
                SRR2589044_1.trim.fastq.gz SRR2589044_1un.trim.fastq.gz \
                SRR2589044_2.trim.fastq.gz SRR2589044_2un.trim.fastq.gz \
                SLIDINGWINDOW:4:20 MINLEN:25 ILLUMINACLIP:NexteraPE-PE.fa:2:40:15
```

```output
TrimmomaticPE: Started with arguments:
 SRR2589044_1.fastq.gz SRR2589044_2.fastq.gz SRR2589044_1.trim.fastq.gz SRR2589044_1un.trim.fastq.gz SRR2589044_2.trim.fastq.gz SRR2589044_2un.trim.fastq.gz SLIDINGWINDOW:4:20 MINLEN:25 ILLUMINACLIP:NexteraPE-PE.fa:2:40:15
Multiple cores found: Using 2 threads
Using PrefixPair: 'AGATGTGTATAAGAGACAG' and 'AGATGTGTATAAGAGACAG'
Using Long Clipping Sequence: 'GTCTCGTGGGCTCGGAGATGTGTATAAGAGACAG'
Using Long Clipping Sequence: 'TCGTCGGCAGCGTCAGATGTGTATAAGAGACAG'
Using Long Clipping Sequence: 'CTGTCTCTTATACACATCTCCGAGCCCACGAGAC'
Using Long Clipping Sequence: 'CTGTCTCTTATACACATCTGACGCTGCCGACGA'
ILLUMINACLIP: Using 1 prefix pairs, 4 forward/reverse sequences, 0 forward only sequences, 0 reverse only sequences
Quality encoding detected as phred33
Input Read Pairs: 1107090 Both Surviving: 885220 (79.96%) Forward Only Surviving: 216472 (19.55%) Reverse Only Surviving: 2850 (0.26%) Dropped: 2548 (0.23%)
TrimmomaticPE: Completed successfully
```

:::::::::::::::::::::::::::::::::::::::  challenge

## Exercise

Use the output from your Trimmomatic command to answer the
following questions.

1) What percent of reads did we discard from our sample?
2) What percent of reads did we keep both pairs?


::::::::::::::::::::::::::::::::::::::::::::::::::

You may have noticed that Trimmomatic automatically detected the
quality encoding of our sample. It is always a good idea to
double-check this or to enter the quality encoding manually.

We can confirm that we have our output files:

```bash
$ ls SRR2589044*
```

```output
SRR2589044_1.fastq.gz       SRR2589044_1un.trim.fastq.gz  SRR2589044_2.trim.fastq.gz
SRR2589044_1.trim.fastq.gz  SRR2589044_2.fastq.gz         SRR2589044_2un.trim.fastq.gz
```

The output files are also FASTQ files. It should be smaller than our
input file, because we have removed reads. We can confirm this:

```bash
$ ls SRR2589044* -l -h
```

```output
-rw-rw-r-- 1 dcuser dcuser 124M Jul  6 20:22 SRR2589044_1.fastq.gz
-rw-rw-r-- 1 dcuser dcuser  94M Jul  6 22:33 SRR2589044_1.trim.fastq.gz
-rw-rw-r-- 1 dcuser dcuser  18M Jul  6 22:33 SRR2589044_1un.trim.fastq.gz
-rw-rw-r-- 1 dcuser dcuser 128M Jul  6 20:24 SRR2589044_2.fastq.gz
-rw-rw-r-- 1 dcuser dcuser  91M Jul  6 22:33 SRR2589044_2.trim.fastq.gz
-rw-rw-r-- 1 dcuser dcuser 271K Jul  6 22:33 SRR2589044_2un.trim.fastq.gz
```

We have just successfully run Trimmomatic on one of our FASTQ files!
However, there is some bad news. Trimmomatic can only operate on
one sample at a time and we have more than one sample. The good news
is that we can use a `for` loop to iterate through our sample files
quickly!

We unzipped one of our files before to work with it, let's compress it again before we run our for loop.

```bash
gzip SRR2584863_1.fastq 
```

```bash
for infile in *_1.fastq.gz
do
  base=$(basename ${infile} _1.fastq.gz)
  trimmomatic PE ${infile} ${base}_2.fastq.gz \
               ${base}_1.trim.fastq.gz ${base}_1un.trim.fastq.gz \
               ${base}_2.trim.fastq.gz ${base}_2un.trim.fastq.gz \
               SLIDINGWINDOW:4:20 MINLEN:25 ILLUMINACLIP:NexteraPE-PE.fa:2:40:15 
done
```

Go ahead and run the for loop. It should take a few minutes for
Trimmomatic to run for each of our six input files. Once it is done
running, take a look at your directory contents. You will notice that even though we ran Trimmomatic on file `SRR2589044` before running the for loop, there is only one set of files for it. Because we matched the ending `_1.fastq.gz`, we re-ran Trimmomatic on this file, overwriting our first results. That is ok, but it is good to be aware that it happened.

```bash
$ ls
```

```output
NexteraPE-PE.fa               SRR2584866_1.fastq.gz         SRR2589044_1.trim.fastq.gz
SRR2584863_1.fastq.gz         SRR2584866_1.trim.fastq.gz    SRR2589044_1un.trim.fastq.gz
SRR2584863_1.trim.fastq.gz    SRR2584866_1un.trim.fastq.gz  SRR2589044_2.fastq.gz
SRR2584863_1un.trim.fastq.gz  SRR2584866_2.fastq.gz         SRR2589044_2.trim.fastq.gz
SRR2584863_2.fastq.gz         SRR2584866_2.trim.fastq.gz    SRR2589044_2un.trim.fastq.gz
SRR2584863_2.trim.fastq.gz    SRR2584866_2un.trim.fastq.gz
SRR2584863_2un.trim.fastq.gz  SRR2589044_1.fastq.gz
```

:::::::::::::::::::::::::::::::::::::::  challenge

## Exercise

We trimmed our fastq files with Nextera adapters,
but there are other adapters that are commonly used.
What other adapter files came with Trimmomatic?



## Solution Output

```output
NexteraPE-PE.fa  TruSeq2-SE.fa    TruSeq3-PE.fa
TruSeq2-PE.fa    TruSeq3-PE-2.fa  TruSeq3-SE.fa
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

We have now completed the trimming and filtering steps of our quality
control process! Before we move on, let's move our trimmed FASTQ files
to a new subdirectory within our `data/` directory.

```bash
$ cd ~/gen711-811/data/untrimmed_fastq
$ mkdir ../trimmed_fastq
$ mv *.trim* ../trimmed_fastq
$ cd ../trimmed_fastq
$ ls
```

```output
SRR2584863_1.trim.fastq.gz    SRR2584866_1.trim.fastq.gz    SRR2589044_1.trim.fastq.gz
SRR2584863_1un.trim.fastq.gz  SRR2584866_1un.trim.fastq.gz  SRR2589044_1un.trim.fastq.gz
SRR2584863_2.trim.fastq.gz    SRR2584866_2.trim.fastq.gz    SRR2589044_2.trim.fastq.gz
SRR2584863_2un.trim.fastq.gz  SRR2584866_2un.trim.fastq.gz  SRR2589044_2un.trim.fastq.gz
```

:::::::::::::::::::::::::::::::::::::::  challenge

:::::::::::::::  solution

## Solution

In your terminal window do:

```bash
$ fastqc ~/gen711-811/data/trimmed_fastq/*.fastq*
```


Then take a look at the html files in your browser.

Remember to replace everything between the `@` and `:` in your scp
command with your AWS instance number.

After trimming and filtering, our overall quality is much higher,
we have a distribution of sequence lengths, and more samples pass
adapter content. However, quality trimming is not perfect, and some
programs are better at removing some sequences than others. Because our
sequences still contain 3' adapters, it could be important to explore
other trimming tools like [cutadapt](https://cutadapt.readthedocs.io/en/stable/) to remove these, depending on your
downstream application. Trimmomatic did pretty well though, and its performance
is good enough for our workflow.



:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- The options you set for the command-line tools you use are important!
- Data cleaning is an essential step in a genomics workflow.

::::::::::::::::::::::::::::::::::::::::::::::::::