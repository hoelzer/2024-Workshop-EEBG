# EEBG 2024: courses & workshops

A practical introduction to 

* Linux for bioinformatics (w/ a focus on `conda`/`mamba`, containers, and workflow management systems)
* Nanopore sequencing, bioinformatics, and (long-read) metagenomics

in context of the the third edition of the Eastern European Bioinformatics and Computational Genomics School ([EEBG 2024](https://www.eebgschool.org/)), Suceava, Romania. 

## Schedule links for the workshop

### Linux, basic Bash, Conda, Git, containers and WMS 
* [2024-07-09 - Tuesday: Welcome, Linux re-cap, container, and WMS](#0)  

### Nanopore sequencing, long-read bioinformatics, and metagenomics 
* [2024-07-10 - Wednesday: Introduction to Nanopore sequencing and long-read metagenomics](#1)  

## Instructors

* [Martin Hölzer](https://github.com/hoelzer), Genome Competence Center, Robert Koch Institute, Germany

## Schedule

> All events are held at [Universitatea „Ștefan cel Mare” din Suceava](https://usv.ro/)

### <a name="0"></a> Tuesday, 9th July 2024
| Time        | Welcome, Linux re-cap, container & WMS |
| --          | --               |
| 11:00-11:15 | Welcome, [introduction](day-welcome-linux-container-wms/general.md) |
| 11:15-12:00 | [Linux re-cap](day-welcome-linux-container-wms/linux.md) |
| 12:00-13:00 | Lunch break |
| 13:00-14:00 | Another lecture (Tatiana Tatarinova, University of la Verne, USA) |
| 14:00-15:00 | [Container & WMS](day-welcome-linux-container-wms/container-wms.md) |
| 15:00-15:45 | [Hands-on & demo](day-welcome-linux-container-wms/hands-on.md) |
| 15:45-16:00 | Wrap-up & questions |

### <a name="1"></a> Wednesday, 10th July 2024
| Time        | Nanopore & metagenomics read classification |
| --          | --               |
| 09:00-09:30 | [Nanopore intro](day-nanopore/README.md) |
| 09:30-09:45 | [Nanopore basecalling & data formats](day-nanopore/README.md) |
| 09:45-10:00 | [Nanopore QC](day-nanopore/README.md) |
| 10:00-11:00 | [Hands-on & demo (nanopore)](day-nanopore/hands-on-metagenomics.md) |
| 11:00-12:00 | [Metagenomic read classification](day-metagenomic-classification/README.md) |
| 12:00-13:00 | Lunch break |
| 13:00-13:45 | [Hands-on & demo (metagenomics)](day-metagenomic-classification/hands-on.md) |
| 13:45-14:00 | Wrap-up & questions |

## Additional material

Because we only had limited time, you can find additional material and hands-on examples from previous workshops here:

* [A practical introduction to long-read bioinformatics on the flu virus, Namibia, 2024](https://github.com/rki-mf1/2024-Workshop-Namibia)
* [A practical introduction to short-read bioinformatics for bacterial genome reconstruction, Madagascar IGS workshop, 2023](https://github.com/rki-mf1/2023-Workshop-Madagascar-IGS)
* [A practical introduction to long-read metagenomics, GHPP "Strengthening Genomic Surveillance" workshop at RKI, 2023](https://github.com/rki-mf1/2023-GHPP-SGS-Nanopore-Metagenomics)

## Acknowledgement

This course material is partly based on the following resources & contributions from great people (no specific order):

* Martin Hoelzer, RKI MF1, content about Linux, container, Nextflow, sequencing, genomic surveillance & glueing everything together
* Sebastian "Raverjay" Krautwurst, FSU Jena, some Linux and ONT content
* Stephan Fuchs, RKI MF1, some Linux and Assembly content
* Matt Huska, RKI MF1, automatic test script for all md code blocks using [codedown](https://github.com/earldouglas/codedown) and general help
* Workshop structure inspired by [https://github.com/cinemaparis/2023](https://github.com/cinemaparis/2023)
* Some ONT intro slides from Josh Quick, [original](https://github.com/cinemaparis/2023/blob/main/day1-Tuesday/slides-Quick.pdf)
* Nanopore long-read bioinformatics tutorial from [timkahlke](https://timkahlke.github.io/LongRead_tutorials)
* Hugues Richard, RKI MF1, content about metagenomics
* ... and for sure many other great scientists I forgot to mention (please send a PR!)