# Workshop - Linux re-cap

## Short Linux and bash re-cap

* Linux/Bash basics + conda setup ([slides](https://docs.google.com/presentation/d/14xELo7lDbd-FYuy144ZDK1tV_ZBdBYun_COelrKYWps/edit?usp=sharing))
* Another good resource: [Introduction to the UNIX command line](https://ngs-docs.github.io/2021-august-remote-computing/introduction-to-the-unix-command-line.html)
* Cheat sheet for Bash: [github.com/RehanSaeed/Bash-Cheat-Sheet](https://github.com/RehanSaeed/Bash-Cheat-Sheet)
* An [online linux terminal to play with    ](https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192)

### Small Cheat Sheet:
```bash
# Print your user name
echo $USER
# change directory to your user home directory (all of these are the same)
cd /home/$USER
cd $HOME
cd ~  # <- the shortest version, I like this one
# show content of current directory
ls
# make a new directory called 'myfolder'
mkdir myfolder
# generate an empty file
touch some-file.txt
# show full content of a file
cat some-file.txt
# show first 10 lines of a file
head some-file.txt
# generate three emtpy files
touch file1.txt file2.txt file3.txt
# concatenate the content of files into a new file
cat file1.txt file2.txt file3.txt > file123.txt
# ... or via a wild card
cat file?.txt > file123.txt
# compress a file 
gzip file123.txt # this will generate a compressed file 'sample.fastq.gz'
# uncompress a gz file
gunzip file123.txt.gz


# make conda/mamba environment and activate it
mamba create -n nanoplot
conda activate nanoplot
mamba install nanoplot
# Attention, the above command will create a folder 'nanoplot' in your default path, e.g. ~/miniconda3/envs
# However, you can also specify any other folder, and which we will also do in the training:
mamba create -p envs/nanoplot
conda activate envs/nanoplot
mamba install nanoplot
# run a program
NanoPlot --help
```

Next: [Container & WMS](container-wms.md)
