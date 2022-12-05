# Testing reform 

## Web tool 

Replicated AB's issue using web tool. Got the same error and was not listed on their GitHub issues page. 

## Commandline tool 

Downloaded the reform.py script from their GitHub page. 
```
git clone https://github.com/gencorefacility/reform 
```

Install biopython:
```
sudo pip install biopython
```

Downloaded the reference fasta and gff3 files from Ensembl's ftp site
```
wget https://ftp.ensembl.org/pub/release-108/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.chromosome.X.fa.gz
wget https://ftp.ensembl.org/pub/release-108/gff3/homo_sapiens/Homo_sapiens.GRCh38.108.chromosome.X.gff3.gz
```

Unzipped the fasta file 
```
gunzip Homo_sapiens.GRCh38.dna.chromosome.X.fa.gz
```

Ran the command
```
python3 reform.py 
  --chrom=X \
  --upstream_fasta ./Inputs/Upstream_X_3kb.fasta \
  --downstream_fasta ./Inputs/Downstream_X_3kb.fasta \
  --in_fasta=./Inputs/CMTX3_SV_FASTA.fasta \
  --in_gff=./Inputs/Chr_R4_R5_edit_Insert_chr8_GTF_Columns.gtf \
  --ref_fasta=./Inputs/Homo_sapiens.GRCh38.dna.chromosome.X.fa \
  --ref_gff=./Inputs/Homo_sapiens.GRCh38.108.chromosome.X.gff3.gz
  ```

  Got error: 
  ```
  No valid position specified, checking for upstream and downstream sequence
Removing nucleotides from position 140420783 - 140420811
Proceeding to insert sequence 'CMTX3_SV_Sequence' from ./Inputs/CMTX3_SV_FASTA.fasta at position 140420783 on chromsome X
New fasta file created:  Homo_sapiens.GRCh38.dna.chromosome.X_reformed.fa
Preparing to create new annotation file
** ERROR: in_gff file does not have 9 columns, it has 1
['NEW havana gene 41113 43561 . + . gene_id "ENSG00000255456"; gene_version "1"; gene_source "havana"; gene_biotype "lncRNA";\n']
```