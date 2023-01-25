# Generating the updated CMTX3 hg38 fasta 

## Reference assembly choices 

Depending on the species you're working on you will either have endless choices or no choice when it comes to which type of genome build available to you. Reference genome assemblies for Hg38 are available through NCBI, Ensembl, UCSC, and human reference genome consortia. For current [Ensembl references](https://ftp.ensembl.org/pub/release-108/fasta/homo_sapiens/dna/) there are a few additional choices: 

First level of choice is about genome masking. Repetetive and low-complexity sequences can be challenging to work with for search and mapping algorithms (like BWA) which look for unique patterns that match in the genome and sequence reads. Low-complexity sequences composed of one or a few nucleotides (like 'aaaataaaaaaat') appear frequently across the genome, so pairing reads to these regions of the genome can be challenging. Masking these regions to varying degrees allows some tools/algorithms to avoid these regions or treat them accordingly. 

* **dna:** (unmasked) is the right choice for you and in most instances where you don't want to lose any information. You can always perform filtering after mapping.  
* **dna_rm:** (repeat masked) indictates bases comprising repetivite and low-complexity regions have been converted to N bases so they cannot be read by a tool. This is generally not recommended for your purposes. 
* **dna_sm:** (soft masked) indicates these repetivite and low-complexity regions have been indicated in the fasta using lower-case letters. However, tools like BWA do not take soft-masking into consideration, so theres no benefit to using the dna_sm over the dna assembly. 

Second level of choice for whole assembly:
* **primary_assembly:** single reference base per position.
* **toplevel:** includes haplotype information.
* **alt:** contains alternate contigs which have implications for variant calling in variable regions of the genome like the HLA locus.

## Making the CMTX3 fasta 

Extract the chr8 sequence from the downloaded and gunzipped [chr8 fasta file](https://ftp.ensembl.org/pub/release-108/fasta/homo_sapiens/dna/) with the following code. You will need to run this as an interactive session if you are working on Artemis as it'll consume more memory than is available on the login nodes. To run an interactive session: 

```bash
# start interactive session 
qsub -P <DashR project name> -I 

# it'll automatically place you in $HOME so you will need to move back to where you were working
cd /path/to/workingDirectory 
```

Then print a substring of the whole file by first concatenating the lines using `tr` which removes newlines. Then run your `awk` command on the whole file: 

```bash
cat Homo_sapiens.GRCh38.dna.chromosome.8.fa | tr -d '\n' | awk '{print substr($0,start_pos,length)}'
```


## Making custom assembly fasta 

To make a custom reference fasta with autosomes from the hg38 assembly, you can run the following script to download all autosome fasta files (just check all the file names are correct):

```bash
#!/bin/bash

# run this script from the directory you want to store the reference assembly in

chromosomes=(1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22)

# download all unmasked chromosome .fasta files individually 
for chromosome in ${chromosomes[@]}; do

  curl "https://ftp.ensembl.org/pub/release-108/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.chromosome.${chromosome}.fa.gz" \
    | zcat >> hg38.fa 
  
done

# add modified chrX.fasta to end of modified_hg38.fa 
cat hg38.fa CMTX3_chrX.fasta > modified_hg38.fa
```

## Index custom (or any) fasta 

You will likely also need to index the fasta file for using it in downstream steps. Using an index file enables efficient querying of regions within the big reference sequences. Different tools require different indexes but some of the most common types are:

* samtools index: `.fai`
* bwa index: `.amb`, `.ann`, `.bwt`, `.pac`, `.sa`
* gatk sequence dictionary: `.dict`

Check which indexes you need (I assume you'll definitely need the samtools and bwa indexes). You can generate them using our [IndexReferenceFasta Nextflow](https://github.com/Sydney-Informatics-Hub/IndexReferenceFasta-nf) workflow. Nextflow is already installed on Nimbus, so you can use this workflow to generate all index files above. To create bwa and samtools indexes (and not the gatk reference), follow instructions in the read me for downloading the code and run: 

```bash
nextflow run main.nf /path/to/modified_hg38.fa --bwa --samtools 
```


