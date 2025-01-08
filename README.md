# amr-bioinfo-demo
## SSH
* check small step method https://scinet.usda.gov/guides/access/ssh-login
```
ssh atlas-login
```
## WEBINAR DIRECTORY TO STORE FILES ASSOCIATED WITH THIS DEMO
```
# a hashtag means 'everything after is a comment; do not run'
wd=$(pwd) #set a working directory parameter for your session
mkdir webinar #make directory
```
```
cd webinar #change directory
```
## PIPELINE AND DATA
```
mkdir data pipeline #make two directories called data and pipeline
```
### get pipeline. https://github.com/Phylloxera/GEA-dev
```
cd pipeline; curl -L -O https://github.com/Phylloxera/GEA-dev/releases/latest/download/geacont.tgz #separate two commands with a semicolon.
```
```
tar -xzvf geacont.tgz -C $HOME/software/ #Extract the container to your $HOME/software directory
```
```
module load apptainer #add apptainer to your environment https://scinet.usda.gov/guides/software/modules
```
```
apptainer exec $HOME/software/geacont cp /usr/local/src/GEAbash $HOME/software/ #Extract the bash script to $HOME/software
```
```
cd ../data; mkdir ecoli1; cd ecoli1 #.. is shorthand for up one directory level
```
### retrieve 3 ecoli assemblies. forward slash allows wrapping 1 command over multiple lines
```
curl -L -O https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/029/433/975/GCF_029433975.1_ASM2943397v1/GCF_029433975.1_ASM2943397v1_genomic.fna.gz \
--next -L -O \
https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/029/434/035/GCF_029434035.1_ASM2943403v1/GCF_029434035.1_ASM2943403v1_genomic.fna.gz \
--next -L -O \
https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/029/434/335/GCF_029434335.1_ASM2943433v1/GCF_029434335.1_ASM2943433v1_genomic.fna.gz 
```
```
gunzip * #extract the data.
```
```
cd ..; mkdir senterica1; cd senterica1
```
### retrieve 4 senterica assemblies
```
curl -L -O https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/032/700/165/GCF_032700165.1_ASM3270016v1/GCF_032700165.1_ASM3270016v1_genomic.fna.gz \
--next -L -O \
https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/032/907/125/GCF_032907125.1_ASM3290712v1/GCF_032907125.1_ASM3290712v1_genomic.fna.gz \
--next -L -O \
https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/210/855/GCF_000210855.2_ASM21085v2/GCF_000210855.2_ASM21085v2_genomic.fna.gz \
--next -L -O \
https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/022/165/GCF_000022165.1_ASM2216v1/GCF_000022165.1_ASM2216v1_genomic.fna.gz
```
```
gunzip *
```
## PREPARE TO RUN THE GEA PIPELINE
```
cd $wd/webinar
```
## RUN THE PIPELINE
* an additional required argument for GEA pipeline version 1 is `-I` since input type is contigs
* additional required arguments for running on atlas (all passed to sbatch https://scinet.usda.gov/guides/use/slurm) are
* `-q` queue (also called partitions) https://www.hpc.msstate.edu/computing/atlas/
* `-a` account The same as your scinet project name https://scinet.usda.gov/guides/data/storage#project-directories
* `-c` cores The number of cores per node on atlas is lower than the pipeline default https://scinet.usda.gov/guides/use/resource-allocation#allocation-of-cores
* does not have a copy icon because of user specific parameters.

``
$HOME/software/GEAbash -i $wd/webinar/data/ecoli1,$wd/webinar/data/senterica1 -I contigs -q atlas -a sandbox -c 48
``
* run the pipeline on the two folders
* record batch number and time
* available input types are `paired-fqgz` (default), `contigs`, 6 long read types: `nano-raw`, `nano-corr`, `nano-hq`, `pacbio-raw`, `pacbio-corr`, `pacbio-hifi` (unassembled long read sequencing in FASTA or FASTQ format, gz compressed or uncompressed)
## GET INFORMATION FROM THE RUN
``
scontrol show job <job id> #record run time and jobstate
``
```
nano slurm-*.out #the out file logs the console outputs of the pipeline
#record 2 resfinder lines, 1 mlst line & terminal message
```
## LOOK AT METADATA
```
nano md_cols.R #man nano
```
### cut and paste Rscript into file md_cols.R
```
#!/usr/bin/env Rscript

args <- commandArgs(trailingOnly = T); if (length(args) != 2) {stop("This script requires a 2 column index arguments; exiting", call. = F)}
first_column <- args[1]; last_column <- args[2]
md <- read.delim("results/metadata.txt", row.names = NULL)
md[1:7, first_column:last_column]
```
### ctrl + X to close file md_cols.R
### print column sets from the result to the console
```
apptainer exec $HOME/software/geacont Rscript md_cols.R 1 4 #columns 1 through 4
```
```
apptainer exec $HOME/software/geacont Rscript md_cols.R 13 14
```
```
apptainer exec $HOME/software/geacont Rscript md_cols.R 15 16
```
