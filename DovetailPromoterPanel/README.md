# Processing and analysis of Dovetail Genomics Promoter Panel

Following documentaion [here](https://dovetail-capture.readthedocs.io/en/latest/index.html). 

## [Alignment (fastq to bam)](https://dovetail-capture.readthedocs.io/en/latest/fastq_to_bam.html)

* Dovetail user guide provides long, piped command comprising multiple steps
* This requires samtools, pairtools, and bwa to be installed 
* For sake of reproducibility, will be using Singularity to run existing containerised tools 
* Singularity can run on HPC 
* Need to break command up into different steps and wrap with Singularity syntax
* FYI in the future the open source version of Singularity will be called Apptainer 

### Customising your environment before you begin  
  
By default, Singularity will set up a directory called `.singularity` in your $HOME directory. This contains a cache directory to make downloading and building images faster, a temporary directory which needs to be large enough to hold a whole singularity image file (SIF). These locations can be configured by setting SINGULARITY_TMPDIR and SINGULARITY_CACHEDIR to desired paths. Set the following environmental variables within the script before running:   

```
mkdir -p /scratch/er01/gs5517/singularity/{cache,tmp}
export SINGULARITY_CACHEDIR="/data/singularity/cache"
export SINGULARITY_TMPDIR="/data/singularity/tmp"
```

### Finding and downloading a container 

Recommend going through [these materials](https://pawseysc.github.io/singularity-containers/) from Pawsey before you begin. There are multiple repopsitories for containers, but I recommend [quay.io](https://quay.io/):

**1. Search for a tool** 

Go to [quay.io](https://quay.io) and search for a tool using search box in top right corner. You don't need to register to use quay.io this way. It will produce a list of container options, opt for a container that is recently maintained. Preference biocontainers if possible. They have requirements for submission (maintenance, documentation, metadata, security) that make them a more reliable source of bioinformatics containers.   

* Example for bwa: https://quay.io/repository/biocontainers/bwa

**2. Choose a container and collect tag**

Select a repository source, nagivate to the tag tab on the left side of the page, and click on download button on the right side of the page for a tag of your choosing. Tags differ between container versions. Check the tool version you're downloading is correct, avoid tags that are unsupported or unable to scan or show a bug/security vulnerability. Select `image source: Docker Pull (by tag)` and copy the path provided. We will use Singularity, not Docker to download the container. Using the `latest` tag, while quick, can lead to issues with reproducibility of your workflow.   

> **_NOTE:_**  Generally it is best to use the most recent version of a tool if you are not required to run a specific version. 

**3. Download and run the container**

You can choose to first download the container to your machine and then run it or you can just download and run it in a single command. To download a container, use the Singularity pull command: 
```
singularity pull docker://quay.io/biocontainers/bwa:0.7.3a--h7132678_7
```

To download and run the container in a single command, just use the link provided by the `Fetch tag` box in quay.io. You can check this works by running the bwa help command:
```
singularity run -B /data \
    docker://quay.io/biocontainers/bwa:0.7.3a--h7132678_7 \
    bwa  
```

> **_NOTE:_**  Singularity is capable of running both Singularity and Docker containers. Docker is the most popular container tool, currently so most widely available. To run a Docker container with Singularity you will need to add `docker://` to the front of the link.

## Alignment script 

Suggest some edits to the script:
```
#!/bin/bash

# script for running the BWA mem step of Dovetail's Promoter Panel Capture Analysis Pipeline 

# define bind paths 
genome=/path/to/ref/genome/directory 
fastq=/path/to/reads/directory 
output=/path/to/output/directory 

# define variables 
ref_fasta=/path/to/ref/fasta (e.g. /data/RefGenomes/Control_hg38/hg38.fa)
read1=/path/to/read1 (e.g. /data/RawFASTQ/DTG-PPC-006a_1.fastq.gz)
read2=/path/to/read2 (e.g. /data/RawFASTQ/DTG-PPC-006a_2.fastq.gz)
sample=<sample info> (e.g. BB1) # for CMTX3 sample, remember to include info on which variation of the 3 ref genomes you are using

# run bwa 
singularity exec -B ${genome} -B ${fastq} -B ${output} \
docker://quay.io/biocontainers/bwa:0.7.3a--h7132678_7 \
    bwa mem -5SP -T0 -t16 \
    ${ref_fasta} ${read1} ${read2} \
    -o  ${output_folder}/${sample}_aligned.sam
``` 

> **_NOTE_** See [here](https://carpentries-incubator.github.io/singularity-introduction/03-singularity-shell/index.html) for differences between `singularity run` and `singularity exec`

## Useful resources on running containers with Singularity 

* [Pawsey's Singularity containers workshop](https://pawseysc.github.io/singularity-containers/)
* [Singularity user guide](https://docs.sylabs.io/guides/3.5/user-guide/introduction.html)
* [Yale centre for research computing materials](https://docs.ycrc.yale.edu/resources/online-tutorials/#singularity-apptainer)
* [Carpentries Singularity course](https://carpentries-incubator.github.io/singularity-introduction/)

