# Workshop: Long-read Nanopore Metagenomics, Data Formats & Quality Control

**Note**: If internet connection is slow, we can also distribute the example data via an USB stick (ask your instructor ;). 

**General**: We will usually start with an "Hands-on" part that will guide you through some of the content from the lectures. Then, there is an "Excercise" part with more open questions. 

**Example data**: We will work with one downsampled metagenomics long-read example data set to showcase different usecases. 

:bulb: Think about a good and descriptive folder and file structure when working on the data!

## How to connect to the Linux VM

**First thing:** To do the hands-on work directly, you need a running Linux system (or terminal on a Mac, or "Windows Subsystem for Linux"), and decent disk space (~20 GB) and RAM (~8 GB).

**It might also be a good idea to work on the USV virtual machines!** (remember the slides that were circulated earlier!)

```bash
# you can use this slightly modified command to connect w/ X11 display forwarding
# depending on your local machine, this should work directly...
ssh -X fmsb@<ip-adress-you-choose>
```

## Create a folder for the hands-on work

Below are just example paths, you can also adjust them and use other folder names! Assuming you are on a Linux system on a local machine (laptop, workstation):

```sh
# Switch to a path on your system where you want to store your data and results
cd ~
# Create new folder fo workshop (if not already done)
mkdir -p nanopore-workshop
cd nanopore-workshop
```

It's always important that you keep a clean and descriptive folder structure when doing bioinformatics or data science in general. Let's start with creating a project directory. First, change to some location on your file system where you want to store the data and create the project folder, then:

**Attention: we will always start from this folder doing the analysis! Double check, e.g. via `pwd`, if you are in the correct folder on your system!**

## Install and use analysis tools

- **Note**: Bioinformatics tools are regulary updated and input parameters might change (use `--help` or `-h` to see the manual for a tool!)
- Install most of them into our environment
    - we will already install many tools that we will use over the next days!
- **if you did that already before and have the environment, skip this**

```bash
mkdir -p envs
# if not already done!
mamba create -y -p envs/qc nanoplot filtlong minimap2 samtools igv
mamba activate envs/qc
# test
NanoPlot --help
minimap2 --version
```

**Reminder: You can also install specific versions of a tool!**

- important for full reproducibility
- e.g. `mamba install minimap2==2.26`
- per default, `mamba` will try to install the newest tool version based on your configured channels and system architecture and dependencies to other tools

## Get example long-read Nanopore data of a mock community

We will use the [ZymoBIOMICS Microbial Community Standard II, Log Distribution (CSII)](https://www.zymoresearch.de/products/zymobiomics-microbial-community-dna-standard-ii-log-distribution) as example. The CSII is a mixture of the genomic DNA of eight bacterial (three Gram-negative and five Gram-positive) and two fungal strains covering a wide range of abundances, with the following theoretical composition: 

* _Listeria monocytogenes_ - 89.1%
* _Pseudomonas aeruginosa_ - 8.9%
* _Bacillus subtilis_ - 0.89%
* _Saccharomyces cerevisiae_ - 0.89%
* _Escherichia coli_ - 0.089%
* _Salmonella enterica_ - 0.089%
* _Lactobacillus fermentum_ - 0.0089%
* _Enterococcus faecalis_ - 0.00089%
* _Cryptococcus neoformans_ - 0.00089%
* _Staphylococcus aureus_ - 0.000089%

We isolated DNA from the Zymo community mix yielding 660 ng/~73 µl recovered with EtOH (219 µl) precipitation + 3M NaOAc (1/10 vol ~29.3 µl) in 13 µl H2O. We used the **ONT Rapid Barcoding Kit** without bead step (SQK-RBK004) for library preparation, which resulted in 11 µl DNA input for a "standard" MinION flow cell (~400 ng input). The prepared library was sequenced on a **MinION with an R9.4.1 flow cell** (FLO-MIN106). A laptop computer with an Intel Core i5-10400H CPU and NVIDIA GeForce RTX 5000, 16 GB GDDR6 GPU was used for basecalling using MinKNOW (v22.08.4) with the **Super accurate basecalling model (SUP)**. The used **barcode was 01**.

After basecalling, we combined all `fastq_pass` reads yielding a 4.5 GB FASTQ file. For this workshop, we downsampled the FASTQ file down to 10% of the oiginal reads (345 MB) using the following command (**you dont have to do that!**):

```bash
## JUST FOR INFORMATION: downsampling was used to reduce the size of the example data set, dont run this! I did this for you!
#mamba create -y -p envs/seqkit seqkit
#conda activate envs/seqkit
#zcat barcode01.fastq.gz | seqkit sample -p 0.1 -o zymo-2022-barcode01-perc10.fastq.gz
```

The downsampled FASTQ file can be downloaded [here](https://osf.io/buajf) or via this command:

```bash
mkdir reads
cd reads
wget https://osf.io/buajf/download -O zymo-2022-barcode01-perc10.fastq.gz
cd ..
```

Let's download the example data and have a look on the FASTQ file! We placed it in a `reads` folder in your working directory! This should then look like this:

```bash
nanopore-workshop/reads/zymo-2022-barcode01-perc10.fastq.gz
```

## Quality control (NanoPlot)

```bash
# assuming you are in your working dir: nanopore-workshop and the "qc" environment is activated
NanoPlot -t 4 --fastq reads/zymo-2022-barcode01-perc10.fastq.gz --title "Raw reads" --color darkslategrey --N50 --loglength -f png -o nanoplot/raw
```

[Publication](https://academic.oup.com/bioinformatics/advance-article/doi/10.1093/bioinformatics/bty149/4934939) | [Code](https://github.com/wdecoster/NanoPlot)

### A note on how to investigate output files on a remote server (VM, ...)

In case you are working on a remote Linux server, you can copy files from it to your local machine to open them with a graphical user interface (editor, firefox, image software, ...). For example, use the same login but instead of connecting via `ssh` copy data via `scp`:

```bash
# on your local (!) machine, make a folder where you want to copy stuff to
cd ~
mkdir nanopore-workshop
cd nanopore-workshop
scp -r fmsb@10.110.11.165:/home/fmsb/nanopore-workshop/nanoplot .
```

By that you can keep a copy of your results on your local machine for easier investigation. 


## Read filtering (Filtlong)

```bash
# Note: we use 1 kb as the minimum read length cutoff as an example. For your "real" samples other parameters might be better! Do QC before. 
filtlong --min_length 1000 reads/zymo-2022-barcode01-perc10.fastq.gz > reads/zymo-2022-barcode01-perc10.filtered.fastq

# Check the quality again:
NanoPlot -t 4 --fastq reads/zymo-2022-barcode01-perc10.filtered.fastq --title "Filtered reads" --color darkslategrey --N50 --loglength -f png -o nanoplot/filtered
```

[Code](https://github.com/rrwick/Filtlong)

## Blast some reads online

Let's check the first read in the data set:

```bash
head -4 reads/zymo-2022-barcode01-perc10.filtered.fastq
```

Copy the nucleotide sequence and [BLAST online](https://blast.ncbi.nlm.nih.gov/Blast.cgi) (using Nucleotide BLAST search).

What do you found? Check also the alignment. Do you see differences (errors?).

## Download reference genomes

We know which species are in our sample. So let's download reference genomes for them!

Also, later, we want to build a custom database for the 10 species included in our mock community, so let's download them from NCBI first:

```bash
# make a folder to store the genome FASTAs
mkdir -p reference-genomes
cd reference-genomes

# Now we download reference genomes for the 10 target species from NCBI
## Bacillus subtilis
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/009/045/GCF_000009045.1_ASM904v1/GCF_000009045.1_ASM904v1_genomic.fna.gz
## Enterococcus faecalis
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/393/015/GCF_000393015.1_Ente_faec_T5_V1/GCF_000393015.1_Ente_faec_T5_V1_genomic.fna.gz
## Escherichia coli
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.fna.gz
## Lactobacillus fermentum
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/029/961/225/GCF_029961225.1_ASM2996122v1/GCF_029961225.1_ASM2996122v1_genomic.fna.gz
## Listeria monocytogenes
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/196/035/GCF_000196035.1_ASM19603v1/GCF_000196035.1_ASM19603v1_genomic.fna.gz
## Pseudomonas aeruginosa
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/006/765/GCF_000006765.1_ASM676v1/GCF_000006765.1_ASM676v1_genomic.fna.gz
## Salmonella enterica
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/006/945/GCF_000006945.2_ASM694v2/GCF_000006945.2_ASM694v2_genomic.fna.gz
## Staphylococcus aureus
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/013/425/GCF_000013425.1_ASM1342v1/GCF_000013425.1_ASM1342v1_genomic.fna.gz
## Saccharomyces cerevisiae
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/146/045/GCF_000146045.2_R64/GCF_000146045.2_R64_genomic.fna.gz
## Cryptococcus neoformans
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/091/045/GCF_000091045.1_ASM9104v1/GCF_000091045.1_ASM9104v1_genomic.fna.gz

# now we extract the FASTA files
gunzip *.fna.gz

# change back to the main working directory
cd ..
```

### Mapping (minimap2)

Now, we want to map the long reads to one of the reference genomes and visualize the data. As an example, we select the _Listeria_ reference genome. 

```bash
# navigat to your main project dir and create a new output folder for mappings
mkdir mapping
minimap2 -ax map-ont reference-genomes/GCF_000196035.1_ASM19603v1_genomic.fna reads/zymo-2022-barcode01-perc10.filtered.fastq > mapping/zymo-2022-listeria.sam
```
[Publication](https://doi.org/10.1093/bioinformatics/bty191) | [Code](https://github.com/lh3/minimap2)

Inspect the resulting SAM file. Check the [SAM format specification](https://samtools.github.io/hts-specs/SAMv1.pdf).

### Visualization of the mapping (IGV)

```bash
# first, we need to convert the SAM file into a sorted BAM file to load it subsequently in IGV
samtools view -bS mapping/zymo-2022-listeria.sam | samtools sort -@ 4 > mapping/zymo-2022-listeria.sorted.bam  
samtools index mapping/zymo-2022-listeria.sorted.bam

# start IGV browser and load the assembly (FASTA) and BAM file, inspect the output
# this might not work on a HPC or VM w/o display forwarding activated!
igv &
```

### Alternative: Visualization of mapping (Tablet)

```bash
# open the GUI (again, might not work on a VM or HPC)
tablet &

# load mapping file as 'primary assembly'
# ->  mapping/zymo-2022-listeria.sam

# load assembly file as 'Reference/consensus file'
# ->  reference-genomes/GCF_000196035.1_ASM19603v1_genomic.fna
```
[Publication](http://dx.doi.org/10.1093/bib/bbs012) | [Code](https://ics.hutton.ac.uk/tablet/)

__Alternative ways to visualize such a mapping are given by (commercial software) such as Geneious or CLC Genomic Workbench.__



## Bonus (and a little detour into containers)

Do basecalling by your own. **Note** this example uses `Guppy`, you can try that first. But the most recent basecaller is `Dorad`. Adapt the commands to also work with `Dorado`. 

**Note that this can be quite tricky and involves deeper Linux knowledge. Usually, you will be fine with the FASTQ and the already basecalled data that comes out of MinKNOW.**

This might not work well on all systems. A good internet connection is needed as well as some basic knowledge in Docker container usage (for Singularity see the paragraph at the end). Also, good hardware and in the best case a GPU are recommended. 

Another nice overview (even though might be slightly outdated) is provided here: [Basecalling with Guppy](https://timkahlke.github.io/LongRead_tutorials/BS_G.html).

From your own sequencing run, you should find raw signal FAST5 data. Re-basecall the signal data to generate a FASTQ output with `guppy`. Chose an appropriate basecalling model (check for details in the MinKNOW report). Unfortunately, `guppy` can not be installed via Conda and is only provided via the ONT community which needs an account. However, the tool can be installed in a so-called Docker container and then run. If you never used [Docker](https://www.docker.com/products/docker-desktop/), here are some [introductory slides](content/container-wms.pdf). A few other good resources:

* [The dark secret about containers in Bioinformatics](https://www.happykhan.com/posts/dark-secret-about-containers/)
* [Container Introduction Training](https://github.com/sib-swiss/containers-introduction-training)

```sh
# Assuming that you have Docker installed and configured
# Get a container image with guppy
docker pull nanozoo/guppy_gpu:6.4.6-1--2c17584

# Navigate to the folder where you have your raw FAST5 data located, e.g. in a folder called 'fast5'. 
# Then start an interactive session from the container and mount the folder you are currently in *into* the container. 
# Otherwise your local files would not be visible from inside of the container.
docker run --rm -it -w $PWD -v $PWD:$PWD nanozoo/guppy_gpu:6.4.6-1--2c17584 /bin/bash

# list the available flowcell and library kits, so you can pick an appropriate basecalling model
guppy_basecaller --print_workflows

# run basecalling, here we're using as an example a model for R10.4.1 flow cell, 
# run with 260 bp/s translocation speed (which was discontinued in summer 2023, now 400 bp/s is default) 
# and the super-acc SUP model
guppy_basecaller –i ./fast5 –s ./guppy_out –c dna_r10.4.1_e8.2_260bps_sup.cfg \
    --num_callers 48 --cpu_threads_per_caller 1

# Attention! This will take ages even on a small FAST5 like in this example. You can also cancel that with "ctrl C". 
# In this example, I ran on 48 cores (num_callers) and for the small example FAST5 this took 24 h for 70% of the reads basecalled.
# You should really run basecalling on a GPU, if possible.  

# Here is an example command using a GPU and some additional qc parameters. 
guppy_basecaller -i ./fast5 -s ./guppy_out_gpu -c dna_r10.4.1_e8.2_260bps_sup.cfg \
    -x auto -r --trim_strategy dna -q 0 --disable_pings

# On the RKI High-Performance Cluster (using Singularity, see below), this command took 2 minutes for basecalling the example data. 
```

### Using Singularity instead of Docker to run Guppy

**Note**: An alternative to Docker is Singularity. The commands are slightly different then. However, on some systems (e.g. a High-performance cluster) it is not possible to use Docker due to permission settings, and then Singularity is an option. Luckily, Docker containers can be easily (and automatically) converted into Singularity. Here are some example commands assuming an HPC with SLURM as the job scheduler (like at the RKI):

```sh
# Get the Docker image and convert to Singularity
singularity pull --name guppy_gpu-6.4.6-1--2c17584.img docker://nanozoo/guppy_gpu:6.4.6-1--2c17584

# Start an interactive session on a GPU node
srun --gpus=1 --pty bash -i

# Start the Singularity image with GPU support
singularity run --nv guppy_gpu-6.4.6-1--2c17584.img

# Make sure you are in the correct directory, which contains the fast5/ subdirectory
cd 2023-08-nanopore-workshop-example-bacteria
ls
# output:
# fast5  fastq  minknow_report.html  pycoqc.html  sequencing_summary_small.txt

# Run guppy
guppy_basecaller -i fast5 -s guppy_out_gpu -c dna_r10.4.1_e8.2_260bps_sup.cfg \
    -x auto -r --trim_strategy dna -q 0 --disable_pings

# You should see in the terminal output that GPU access is activated:
# ...
# gpu device:         auto
# ...
```
