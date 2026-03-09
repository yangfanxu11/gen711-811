---
title: Working with Files and Directories
teaching: 30
exercises: 15
---
# Lab4 - 2/13/26
::::::::::::::::::::::::::::::::::::::: objectives

- View, search within, copy, move, and rename files. Create new directories.
- Use wildcards (`*`) to perform operations on multiple files.
- Make a file read only.
- Use the `history` command to view and repeat recently used commands.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How can I view and search file contents?
- How can I create, copy and delete files and directories?
- How can I control who has permission to modify a file?
- How can I repeat recently used commands?

::::::::::::::::::::::::::::::::::::::::::::::::::

### EXERCISE 1: NAVIGATION PRACTICE
Navigate to your untrimmed_fastq directory in one command

cd shell_data/untrimmed_fastq

### EXERCISE 2: WILDCARDS
What would the output look like if the wildcard could *not* be matched? Compare the outputs

ls *fq
ls: cannot access '*fq': No such file or directory

### EXERCISE 3: NAVIGATING PRACTICE
Navigate to your home directory. From there, list the contents of the untrimmed_fastq directory.

cd
ls shell_data/untrimmed_fastq

:::::::::::::::::::::::::::::::::::::::  challenge

## Relative path resolution

Using the filesystem diagram below, if `pwd` displays `/Users/thing`,
what will `ls ../backup` display?

1. `../backup: No such file or directory`
2. `2012-12-01 2013-01-08 2013-01-27`
3. `2012-12-01/ 2013-01-08/ 2013-01-27/`
4. `original pnas_final pnas_sub`

![](fig/filesystem-challenge.svg){alt='File System for Challenge Questions'}

4

## Solution 

1. No: there *is* a directory `backup` in `/Users`.
2. No: this is the content of `Users/thing/backup`,
  but with `..` we asked for one level further up.
3. No: see previous explanation.
  Also, we did not specify `-F` to display `/` at the end of the directory names.
4. Yes: `../backup` refers to `/Users/backup`.

:::::::::::::::::::::::::


### EXERCISE 4: FINDING HIDDEN DIRECTORIES
First navigate to the shell_data directory. There is a hidden directory within this directory. Explore the options for ls to find out how to see hidden directories. List the contents of the directory and identify the name of the text file in that directory.

Hint: hidden files and folders in Unix start with ., for example .my_hidden_directory

What is the hidden file name in the hidden directory?

youfoundit.txt

### EXERCISE 5: HISTORY
Find the line number in your history for the command that listed all the .sh files in /usr/bin. Rerun that command.

[yx1040@ron shell_data]$ !93
cd gen711-811/shell_data
bash: cd: gen711-811/shell_data: No such file or directory

### EXERCISE 6: FILE CONTENTS
Print out the contents of the ~/shell_data/untrimmed_fastq/SRR097977.fastq file. What is the last line of the file?

C:CCC::CCCCCCCC<8?6A:C28C<608'&&&,'$

### EXERCISE 7: PATHS
From your home directory, and without changing directories, use one short command to print the contents of all of the files in the ~/shell_data/untrimmed_fastq directory.

cat gen711-811/shell_data/untrimmed_fastq/*

### EXERCISE 8: LESS (Sequence = TTTTTq)
What are the next three nucleotides (characters) after the first instance of the sequence quoted above?

patterns not found
