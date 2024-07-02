# Workshop: Metagenomic read classification

We will continue with the same example data from [ZymoBIOMICS Microbial Community Standard II, Log Distribution (CSII)](https://www.zymoresearch.de/products/zymobiomics-microbial-community-dna-standard-ii-log-distribution), downsampled to 10% of the total number of reads, that we [downloaded previously](../day-nanopore/hands-on-metagenomics.md). Continue using the length-filtered data set that you should have stored in a folder such as `reads/zymo-2022-barcode01-perc10.filtered.fastq`.

## Kraken2 Install

We want to use Kraken 2 for read classification. Let's install it:

```bash
mkdir -p envs
mamba create -y -p envs/kraken2 -c bioconda kraken2
conda activate envs/kraken2 
```

## Kraken2 Databases

_The following commands are based on this [manual](https://software.cqls.oregonstate.edu/updates/docs/kraken2/MANUAL.html#kraken-2-databases)._

A Kraken2 database is a directory containing at least 3 files:

- `hash.k2d`: Contains the minimizer to taxon mappings
- `opts.k2d`: Contains information about the options used to build the database
- `taxo.k2d`: Contains taxonomy information used to build the database

None of these three files are in a human-readable format. Other files may also be present as part of the database build process, and can, if desired, be removed after a successful build of the database.

In interacting with Kraken 2, you should not have to directly reference any of these files, but rather simply provide the name of the directory in which they are stored. Kraken 2 allows both the use of a **standard** database as well as **custom** databases.

### Build our own custom database for Kraken2

Now, we build a custom database for the 10 species included in our mock community. And one with 9 species skipping _Listeria_.

```bash
# create a folder to store the database(s)
mkdir -p databases/custom
cd databases/custom

# Install a taxonomy. Usually, you will just use the NCBI taxonomy, which you can easily download using
kraken2-build --download-taxonomy --db mock10
# This will download the accession number to taxon maps, as well as the taxonomic name and tree information from NCBI
# 40 GB will be downloaded and multiple files. But we need this to match our target genomes to the NCBI taxonomy.

# Now we add each of the 10 reference genomes to this library. You can do this one by one or using a so-called "for loop"
for GENOME in ../../reference-genomes/*.fna; do
    kraken2-build --add-to-library $GENOME --db mock10
done

# ... and finally we build the database
kraken2-build --build --threads 4 --db mock10

# Now, let's also build a second database but skipping the Listeria genome
# Create a new folder for that extra database
mkdir mock9
# we dont want to download the NCBI taxonomy again, so we "link" it into that new folder
ln -s mock10/taxonomy mock9/

# now we add the genomes again to the mock9 library but skipping the one for Listeria
for GENOME in ../../reference-genomes/*.fna; do
    if [[ $GENOME != "GCF_000196035.1_ASM19603v1_genomic.fna" ]]; then
        kraken2-build --add-to-library $GENOME --db mock9
    fi
done

# and build:
kraken2-build --build --threads 4 --db mock9

# After building a database, if you want to reduce the disk usage of the database, you can use the --clean option for kraken2-build to remove intermediate files from the database directory.
# ATTENTION: this will remove the `taxonomy` folder which saves a lot of disk space but then you have to download it again when building another custom DB
kraken2-build --clean --db mock9
kraken2-build --clean --db mock10

# Inspecting a Kraken 2 Database's Contents
kraken2-inspect --db mock10
```

**Note:** If this does not work, pre-build kraken2 databases of the 10 and 9 species can be downloaded [here](https://osf.io/prq82) and [here](https://osf.io/n8kvx). After download, place them in the correct folder and extract them via `tar zxvf mock10.tar.gz` etc. For example (**only do this if you did not build the databases yourself!**):

```bash
# create a folder to store the database(s)
mkdir -p databases/custom
cd databases/custom

wget https://osf.io/prq82/download -O mock10.tar.gz
wget https://osf.io/n8kvx/download -O mock9.tar.gz

tar zxvf mock10.tar.gz
tar zxvf mock9.tar.gz
```

### Build a standard Kraken 2 database (we will not do that during the course)

To create the standard Kraken 2 database, you can use the following command:

- Replace `$DBNAME` above with your preferred database name/location
- Please note that the database will use approximately 100 GB of disk space during creation!

```bash
kraken2-build --standard --threads 8 --db $DBNAME
```

This will download NCBI taxonomic information, as well as the complete genomes in RefSeq for the bacterial, archaeal, and viral domains, along with the human genome and a collection of known vectors.

If this fails due to network restrictions, try:

```bash
kraken2-build --standard --threads 8 --db $DBNAME --use-ftp
```

### Pre-build Kraken2 databases

It is also possible to just download pre-build databases, which is convenient but take care of the file size!

Here are two databases pre-build on NCBI taxnomoy and sequences:

```bash
mkdir -p databases/pre-build/ncbi
cd databases/pre-build/ncbi
# Standard, Refeq archaea, bacteria, viral, plasmid, human1, UniVec_Core
wget https://genome-idx.s3.amazonaws.com/kraken/k2_standard_20230605.tar.gz

# PlusPF-16, Standard plus Refeq protozoa & fungi capped at 16 GB
wget https://genome-idx.s3.amazonaws.com/kraken/k2_pluspf_16gb_20230605.tar.gz
```

But there are also other databases, such as the Genome-Taxnomy Database ([GTDB](https://gtdb.ecogenomic.org/)) that can be more complete in terms of (uncultured) metagenome-derived species:

```bash
# https://bridges.monash.edu/articles/dataset/GTDB_r89_54k/8956970, e.g. we can download a 16GB capped version of the GTDB r89 54k dereplicated kraken2 database from this resource
# more information: https://github.com/rrwick/Metagenomics-Index-Correction
mkdir -p databases/pre-build/gtdb
cd databases/pre-build/gtdb
wget https://bridges.monash.edu/ndownloader/files/16378274 -O gtdb_r89_54k_kraken2_16gb.tar
```

## Kraken2 classification & visualization of results

To classify a set of sequences, use the `kraken2` command. We will now start classifying the example

```bash
# navigat to your main work dir
# create an output folder
mkdir kraken-results
kraken2 --threads 4 --db databases/custom/mock10 --output kraken-results/mock10.kraken.out --report kraken-results/mock10.kraken.report reads/zymo-2022-barcode01-perc10.filtered.fastq
```

### The (terminal) output

Each sequence (or read) classified by Kraken 2 results in a single line of output. Kraken 2's output lines contain five tab-delimited fields; from left to right, they are:

1. "C"/"U": a one letter code indicating that the sequence was either classified or unclassified.
2. The sequence ID, obtained from the FASTA/FASTQ header.
3. The taxonomy ID Kraken 2 used to label the sequence; this is 0 if the sequence is unclassified.
4. The length of the sequence in bp. In the case of paired read data, this will be a string containing the lengths of the two sequences in bp, separated by a pipe character, e.g. "98|94".
5. A space-delimited list indicating the LCA mapping of each k-mer in the sequence(s). For example, "562:13 561:4 A:31 0:1 562:3" would indicate that:

- the first 13 k-mers mapped to taxonomy ID #562
- the next 4 k-mers mapped to taxonomy ID #561
- the next 31 k-mers contained an ambiguous nucleotide
- the next k-mer was not in the database
- the last 3 k-mers mapped to taxonomy ID #562

### Sample Report Output Format

Kraken 2's standard sample report format is tab-delimited with one line per taxon. The fields of the output, from left-to-right, are as follows:

1. Percentage of fragments covered by the clade rooted at this taxon
2. Number of fragments covered by the clade rooted at this taxon
3. Number of fragments assigned directly to this taxon
4. A rank code, indicating (U)nclassified, (R)oot, (D)omain, (K)ingdom, (P)hylum, (C)lass, (O)rder, (F)amily, (G)enus, or (S)pecies. Taxa that are not at any of these 10 ranks have a rank code that is formed by using the rank code of the closest ancestor rank with a number indicating the distance from that rank. E.g., "G2" is a rank code indicating a taxon is between genus and species and the grandparent taxon is at the genus rank.
5. NCBI taxonomic ID number
6. Indented scientific name

**Which species did we find?**

We can investigate the output files. But also visualize the results, e.g. with Krona

### Krona

```bash
# we install krona also in the kraken2 env
conda activate kraken2
mamba install -c bioconda krona

# when this is done, we need to update the NCBI taxonomy for Krona once:
ktUpdateTaxonomy.sh
```

We need two relevant columns for the Krona plot:

- NCBI Taxonomy ID (-t)
- counts (-m)

In a default Kraken2 report, these are in columns 5 and 3, respectively, so we run the Krona command accordingly:

```bash
mkdir krona-results
ktImportTaxonomy -t 5 -m 3 -o krona-results/mock10.krona.html kraken-results/mock10.kraken.report
```

#### Krona using krakentools (optional)

There is a nice software suite called `krakentools` helping with formatting Kraken 2 output for subsequent analyses. To produce a similar Krona plot like before, we can also do:

```bash
conda activate kraken2
# install krakentools
mamba install -c bioconda krakentools
# convert the kraken2 report output
kreport2krona.py -r kraken-results/mock10.kraken.report -o kraken-results/mock10.kraken.report.krona
# plot
ktImportText kraken-results/mock10.kraken.report.krona -o krona-results/mock10.krona.v2.html
```

### Sankey

Another nice visualization can be done via so-called Sankey plots. We will us a online tool: [Pavian](https://fbreitwieser.shinyapps.io/pavian).

- open the web tool
- upload the `kraken-results/mock10.kraken.report` file
- check the output in the "Results overview" and "Sample" reiters

## EXCERCISE

Now, re-do the Kraken 2 classification and visualizations using the `mock9` database (missing _Listeria_) instead of the `mock10` database.

What do you notice? How many unclassified reads do you have in comparison?

**Note** that you can upload multiple samples in [Pavian](https://fbreitwieser.shinyapps.io/pavian)! That's quite convenient for comparisons!

Now also try a much larger database that does not just have the species we actually know are in our mock community: For example, we can ue the "standard Kraken 2" database comprising Refeq archaea, bacteria, viral, plasmid, human, UniVec\_Core. The database should be in this folder: `databases/pre-build/ncbi/k2_standard_20230605.tar.gz`

Now, we need to extract it:

```bash
cd databases/pre-build/ncbi/
mkdir k2_standard_20230605
tar zxvf k2_standard_20230605.tar.gz -C k2_standard_20230605
# this can take a moment... the database is large
```

Now, we again classify:

```bash
# fist change back to your main project folder, then:
kraken2 --threads 4 --db databases/pre-build/ncbi/k2_standard_20230605 --output kraken-results/ncbi-standard.kraken.out --report kraken-results/ncbi-standard.kraken.report reads/zymo-2022-barcode01-perc10.filtered.fastq
```

Inspect the results. Which species do you find in comparison to the custom build `mock10` database?

Now, try also the pre-build PlusPF-16 database (Standard database **plus** Refeq protozoa & fungi, capped at 16 GB size):

```bash
cd databases/pre-build/ncbi/
mkdir k2_pluspf_16gb_20230605
tar zxvf k2_pluspf_16gb_20230605.tar.gz -C k2_pluspf_16gb_20230605
# again, this can take a moment... the database is still large

# fist change back to your main project folder, then:
cd ../../../
kraken2 --threads 4 --db databases/pre-build/ncbi/k2_pluspf_16gb_20230605 --output kraken-results/ncbi-plusPF16.kraken.out --report kraken-results/ncbi-plusPF16.kraken.report reads/zymo-2022-barcode01-perc10.filtered.fastq
```

What do you find now? **Keep in mind how much impact the reference database always has on your results!**