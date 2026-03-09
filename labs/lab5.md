---
title: "Lab 5"
author: "Jeffrey Miller"
date: "2026 Feb 20"
output: pdf_document
---


### Today
1. Review
2. Permissions
3. Conda environments
4. Running Fastqc
5. Redirection (if there is time)

:::::::::::::::::::::::::::::::::::::::: keypoints

- You can view file permissions using `ls -l` and change permissions using `chmod`.
- The `history` command and the up arrow on your keyboard can be used to repeat recently used commands.
- Conda environments simplify reporducability, dependencies and sharing environments.

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::: review

- You can view file contents using `less`, `cat`, `head` or `tail`.
- The commands `cp`, `mv`, and `mkdir` are useful for manipulating existing files and creating new directories.

### Quality scores. Highest score for a base is 41
Quality encoding: !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJ
                   |         |         |         |         |
Quality score:    01........11........21........31........41

::::::::::::::::::::::::::::::::::::::::::::::::::

*Warm ups: What is the last read in the SRR2584863_1.fastq file? How confident are you in this read?
A: 

How big are your fastqs? (Hint: Look at the options for the ls command to see how to show file sizes.)
- hint, it involves 'ls'. See if you can do it using a relative and absolute path
- another hint: There is an option to make it easy to read the file size. Use one of the two methods to find it
A:



### EXERCISE 5.1
Starting in the shell_data/untrimmed_fastq directory, do the following:

Make sure that you have deleted your backup directory and all files it contains.
Create a backup of each of your FASTQ files using cp. (Note: You’ll need to do this individually for each of the two FASTQ files. We haven’t learned yet how to do this with a wildcard.)
Use a wildcard to move all of your backup files to a new backup directory.
Paste the code you used to do each step between the \'\'\' below:


```
rm -Rf backup/
cp SRR097977.fastq SRR097977_backup.fastq
cp SRR097978.fastq SRR098026_backup.fastq
mkdir backup
mv *_backup.fastq backup/
```

### File Permissions Help
The first part of the output for the `-l` flag gives you information about the file's current permissions. There are ten slots in the
permissions list. The first character in this list is related to file type, not permissions, so we'll ignore it for now. The next three
characters relate to the permissions that the file owner has, the next three relate to the permissions for group members, and the final
three characters specify what other users outside of your group can do with the file. We're going to concentrate on the three positions
that deal with your permissions (as the file owner).

![](fig/rwx_figure.svg){alt='Permissions breakdown'}

Here the three positions that relate to the file owner are `rw-`. The `r` means that you have permission to read the file, the `w`
indicates that you have permission to write to (i.e. make changes to) the file, and the third position is a `-`, indicating that you
don't have permission to carry out the ability encoded by that space (this is the space where `x` or executable ability is stored, we'll
talk more about this later).

### EXERCISE 5.2
Change the permissions on all of your backup files to be write-protected.

```

chmod a-w backup/*
```

How do you know they are write protected?
A:ls -l backup
total 92
-r--r--r--. 1 yx1040 domain users 47552 Mar  6 14:42 SRR097977_backup.fastq
-r--r--r--. 1 yx1040 domain users 43332 Mar  6 14:44 SRR098026_backup.fastq


### EXERCISE 5.3: CONDA ENVIRONMENTS AND PROGRAMS
After loading a conda environment, where is the program 'fastqc' stored?

```
which fastqc

```

### Explore the fastqc output. Which samples failed at least one of FastQC’s quality tests? What test(s) did those samples fail?


:::::::::::::::::::::::::::::::::::::::: keypoints
- Use `which` for commands/programs to see where they are installed
- You can view file permissions using `ls -l` and change permissions using `chmod`.
- The `history` command and the up arrow on your keyboard can be used to repeat recently used commands.
- Explain what a conda environment is, and how to activate and deactivate it

::::::::::::::::::::::::::::::::::::::::::::::::::

unzip
