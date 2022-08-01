# Data for the `Metabarcoding: from Lab to Bioinformatics` course


## Sequence quality control

**QC** directory contains DNA reads obtained using different sequencing instruments - Illumina MiSeq and NovaSeq 6000, MGI-Tech DNBSEQ-G400RS, Oxford Nanopore MinION, and PacBio Sequel II.

## ITS2

**MiSeq_Fungi** directory contains sequences of soil fungi.

**Set01**, **Set02**, and **Set03** are the sample sets for individual projects.

Forward primer `GTGARTCATCGAATCTTTG` (`fITS7`; Ihrmark et al., 2012; [DOI:10.1111/j.1574-6941.2012.01437.x](https://academic.oup.com/femsec/article/82/3/666/492046))<br/>
Reverse primer `TCCTCCGCTTATTGATATGC` (`ITS4`; White et al., 1990; [DOI:10.1016/B978-0-12-372180-8.50042-1](https://www.sciencedirect.com/science/article/pii/B9780123721808500421))<br/>


## 16S

**16S** data (mice fecal microbiome, Kozich et al., 2013; [DOI:10.1128/AEM.01043-13](https://journals.asm.org/doi/10.1128/AEM.01043-13)) could be [downloaded from here](https://mothur.s3.us-east-2.amazonaws.com/wiki/miseqsopdata.zip). See also [`mothur` MiSeq SOP](https://mothur.org/wiki/miseq_sop/).

Amplicons of the V4 region of the 16S rRNA gene were obtained with the following primers (for details, see [here](https://github.com/SchlossLab/MiSeq_WetLab_SOP/blob/master/MiSeq_WetLab_SOP.md)):<br/>
Forward primer `GTGCCAGCMGCCGCGGTAA`<br/>
Reverse primer `GGACTACHVGGGTWTCTAAT`<br/>

**NB!** Primers were already trimmed from the sequences!

## Databases for taxonomy annotation

**DB** directory contains databases for taxonomy annotation.

For ITS, use `UNITE_Fungal_ITS.fa` (Nilsson et al., 2019; [DOI:10.1093/nar/gky1022](https://academic.oup.com/nar/article/47/D1/D259/5146189)). The file was prepared by Andrea Telatin for the [Dadaist2 pipeline](https://quadram-institute-bioscience.github.io/dadaist2/) ([file source](https://github.com/quadram-institute-bioscience/dadaist2/releases/download/v0.7.3/uniref.fa.gz)).

For 16S data, you may use Silva v.138.1 database ([download link](https://zenodo.org/record/4587955); Quast et al, 2013; [DOI:10.1093/nar/gks1219](https://academic.oup.com/nar/article/41/D1/D590/1069277)).
