# Workshop: Linux re-cap, Container, and WMS

## Hands-on

### Linux practices and examples

Here are some example commands and things you can try on your own. Always remember:

* use _auto completion_ as often as possible, you can always use the _tab_/_tabulator_ key to get suggestions for a command you are typing and to auto-complete folder/file names and paths - it's much faster and less error prone! (preventing typos!)
* prevent whitespaces in all folder and file names! You can use whitespaces in general, but it will complicate your work on a Linux system! Use `-`, `_`, etc... instead, e.g. `new-file.txt`
* be always careful when you delete a folder or file! It's not as easy as on windows to get your data back!

Now, open a terminal and try the following commands.

```bash
# when opening a new terminal, you always start in your home directory

# the following command shows the current path you are located (remember the tree-like structure of folders on a linux system!)
pwd

# create a new directory
mkdir testdir

# change into that new directory
cd testdir

# check where you are located now
pwd

# generate a new empty file
touch genome.fasta

# list content of the current directory
ls

# list more details, in a human readable format
ls -lah

# write some content into that file
printf ">Sequence\nATCGTACGTACGTAC\n" > genome.fasta

# check content of the file
cat genome.fasta

# change to your home directory
#     ~ is a short version of /home/$USER
cd ~

# check again the content of the file you created
# now you have to type the full path to find the file! Use auto-complete! Here we use the so-called relative path
cat testdir/genome.fasta

# you can also use the absolute path
cat /home/$USER/testdir/genome.fasta

# Hint: $USER is a so-called variable. To see the content of a variable you can also use echo:
echo $USER

# in $USER your terminal stored the information about the current user running the session. You can also define your own variables, for example you could store the absolute path to your file in a variable for easier re-usage:
GENOME=/home/$USER/testdir/genome.fasta
cat $GENOME

# please notice that we always use a leading $ sign when we want to access the content of a variable! See the difference:
echo GENOME
echo $GENOME

# generate another file
touch genome2.fasta

# copy the file to the test folder
cp genome2.fasta testdir/

# list the content of the test folder
ls -lah testdir/

# remove the original file we just generated in your home dir 
rm genome2.fasta

# is it gone?
ls -lah

# however, remember we copied the file so a copy of the file we just deleted is still in the test folder
ls -lah testdir/
```

### Install mamba (if done already on your machine: skip)

* Mamba is a packaging manager that will help us to install bioinformatics tools and to handle their dependencies automatically
* Mamba works together with the conda package manager, and makes installing packages faster
* You will use the mamba command to create environments and install packages, and conda command for some other package management tasks like configuration and activating environments (yes it can be a bit confusing)
* __Hint:__ You can create as many environments as you want! It is often convenied to have separate environments for separate tasks, pipelines, or even tools 
* In the terminal enter:

```bash
# Switch to a directory with enough space, this can be /scratch on a HPC or 
# your ~ (remember that's short for /home/$USER) on your laptop
cd /home/$USER

# make a new folder called 'workshop'
mkdir workshop

# switch to this folder
cd workshop

# Download mamba installer
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-Linux-x86_64.sh"

# ATTENTION: the space in your home directory might be limited (e.g. 10 GB) and per default conda installs tools into ~/.conda/envs
# Thus, take care of your disk space!
# On the HPC you can take care of this by moving ~/.conda to /scratch and making a symlink from your home directory:
# mv ~/.conda /scratch/dot-conda
# ln -s /scratch/dot-conda ~/.conda

# Run installer
bash Mambaforge-Linux-x86_64.sh
# Use space to scroll down the license agreement
# then type 'yes'
# accept the default install location with ENTER
# when asked whether to initialize mamba type 'yes'

# Now start a new shell or simply reload your current shell via
bash

# You should now be able to create environments, install tools and run them
```

* Set up mamba

```bash
# add repository channels for bioconda
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

* Create and activate a new conda/mamba environment

```bash
# -n parameter to specify the name
mamba create -p envs/workshop

# activate this environment
conda activate envs/workshop

# You should now see (workshop) at the start of each line.
# You switched from the default 'base' environment to the 'workshop' environment.
# Which is placed in a folder envs/workshop
```


### Install and use analysis tools

* **Note**: Bioinformatics tools are regulary updated and input parameters might change (use `--help` or `-h` to see the manual for a tool!)
* Install most of them into our environment
    * we will already install many tools that we will use over the next days!

```bash
mkdir envs
mamba create -y -p envs/qc nanoplot filtlong minimap2 samtools igv
conda activate envs/qc
# test
NanoPlot --help
minimap2 --version
```

__Reminder: You can also install specific versions of a tool!__
* important for full reproducibility
* e.g. `mamba install minimap2==2.26`
* per default, `mamba` will try to install the newest tool version based on your configured channels and system architecture and dependencies to other tools

### Create a folder for the hands-on work

Below are just example paths, you can also adjust them and use other folder names! Assuming you are on a Linux system on a local machine (laptop, workstation):

```sh
# Switch to a path on your system where you want to store your data and results, for example
cd /home/$USER
# Create new folder
mkdir nanopore-workshop
cd nanopore-workshop
```

### Container and WMS (brief intro)

Check the small example at [https://github.com/hoelzer/nf_example](https://github.com/hoelzer/nf_example). Clone the repository using `git`. 

Then investigate the `Dockerfile` and try to build the container image locally using `docker build .`. Remember that you can also give your container image a specific name using the `-t` parameter. 

Install `nextflow`, for example directly from [https://nextflow.io/](https://nextflow.io/) or using `conda` or `mamba`. 

Try to get the little `nextflow` example workflow running. The workflow is using `sourmash` so you either need to install the dependency or provide an available container image, see these [code lines](https://github.com/hoelzer/nf_example/blob/master/main.nf#L14-L18). 
