**ANEQ Homework #1: Practice Importing Data, Demultiplexing Reads, and Denoising**

**Due Feb 12th at midnight**

**Name: Alex Jones**

**For complete credit for this assignment, you must answer all questions and include all commands in your obsidian upload.**

------------------------------------------------------------------------
**Learning Objectives**

1. Practice independently running the first few steps of a Qiime2 workflow We will focus on importing data, demultiplexing, and denoising for homework 1.

2. Practice recording commands and editing code to match your analysis

3. Push to GitHub for credit

------------------------------------------------------------------------


# Project summary

20 adult dairy cattle were sampled (via swabbing) to determine the microbial community composition between 5 different body sites (nasal/skin/udder/fecal/oral).  We want you to determine if and how the microbial composition changes between sites over the course of multiple homework assignments.

We will first begin by copying raw sequencing data from a public folder on Alpine into a new directory (that you create) on your Alpine account.


# Initial Steps

1. Log into Alpine using OnDemand and create a new directory for this new analysis in your _scratch_ directory. Hint: it should look something like: /scratch/alpine/$USER/cow/
2. Move into your new directory using OnDemand
3. Create the following sub-directories using OnDemand: 
	1. slurm
	2. taxonomy
	3. tree
	4. taxaplots
	5. dada2
	6. demux
	7. metadata
	8. core_metrics

4. Download the cow_barcodes.txt and cow_metadata.txt files from Canvas and upload them both to your metadata folder within your new cow directory. So, your filepath should look something like this: /scratch/alpine/$USER/cow/metadata

5. On OnDemand, go to your cow directory and open a new terminal

6. Copy the raw sequencing files from this public folder to **your new folder** using the terminal. Do **not** change the names of these files and folders. Hint: make sure you are in your new cow folder before you run this code (this will copy over the whole folder):  

## Import Data


```
#Check directory:
pwd 

#Check to see if subdirectories are present

ls

#Import files

cp -r /pl/active/courses/2024_summer/maw_2024/raw_reads .
```


#### Working directory
```



```
#### Class reservation
```
sinteractive --reservation=aneq505 --time=01:00:00 --partition=amilan --nodes=1 --
ntasks=2 --qos=normal
```
#### Out of class reservation
```
ainteractive --time=01:00:00 --parition=amilan --notes=1 --ntasks=2 --qos=normal
```
- Time can be adjusted based on how long you think you will need the reservation
for..
### Load Qiime2/ Qiime2 version
- qiime2 analysis was done with qiime2 amplicon version 2024.10
```
module purge
module load qiime2/2024.10_amplicon
```
I then like to put the date I worked on something so for example :
2/1/2026
### Demultiplexing
```
command for demultiplexing
```
- I will then put a note for how everything went ex:
- Demultiplexing completed
- Average quality score from demux summary was 30.
- Will trim at 0 bp.
- Will truncate at 250 bp.
- I will also save the qiime2 .qzv files from analysis and save them in the same
file as my obsidian vault and then link the files below the command (you don't have
to this, I do so i


