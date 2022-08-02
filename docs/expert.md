# "Expert mode" commands


## Docker

To open a shell (command-line interface) of the Docker container (e.g., using `ubuntu` image), run:
```bash
docker run -it -v /my_laptop/data:/container/data ubuntu
```
Parameter `-v` allows mounting (attaching) a directory on a host system into a container. A value that should be passed to the `-v` parameter consists of two parts delimited by a colon symbol: a directory on a host system (e.g., `/my_laptop/data`) and a directory in the container (`/container/data`).

**Windows users:** If your data are located in `C:\Users\koljalg\Downloads`, then specify the path as `/c/Users/koljalg/Downloads`.


## Command line basics

Changing the working directory and listing files:
```bash
# Change the working directory to your home directory
cd ~
cd $HOME   # the same

# Change working directory to `/data`
cd /data

# List files
ls

# List files and display results in a long listing format (each file on a separate row)
ls -l

# List files in a long format, sort by file size
ls -l -S
```

Checking data integrity; Wildcards, parameter expansion, and output redirection:
```bash
# Verify the integrity of the files
cd /data/QC
cat QC.md5
md5sum -c QC.md5

# Calculate MD5 checksums for all `fq.gz` files in the current directory,
# Write the results into `Checksum.md5` file
md5sum *.fq.gz > Checksum.md5

# Calculate MD5 checksums only for two files - `MiSeq_R1.fq.gz` and `MiSeq_R2.fq.gz`
# use parameter expansion
md5sum MiSeq_R{1,2}.fq.gz
```

Matching text patterns, pipe operator:
```bash
# View the contents of the file. To exit, press the q key
less QC.md5

# Look for "MiSeq" string in the file
grep "MiSeq" QC.md5

# Count the number of lines in a file that contains "MiSeq" text
grep "MiSeq" QC.md5 | wc -l
```

More pipes:
```bash
# Find and highlight primer motif in the sequences
zgrep "GTGAATCATCGAATCTTTG" --color=always "MiSeq_R1.fq.gz" | less -R

# Find sequences containing a primer motif,
# Exclude sequences with the other motif,
# Select unique sequences, and write the results into a text file
zgrep "GTGAATCATCGAATCTTTG" "MiSeq_R1.fq.gz" \
   | grep -v "TAAGCGGCGGACT" \
   | sort | uniq \
   > some_sequences.txt
```

## Download example data

If you do not have the `wget` program installed in the container, run:
```bash
apt update && apt install -y wget
```

Sequences obtained with different instruments (for QC):
```bash
mkdir -p QC

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/QC/MiSeq_R{1,2}.fq.gz -P QC

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/QC/NovaSeq_R{1,2}.fq.gz -P QC

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/QC/DNBSeq_R{1,2}.fq.gz -P QC

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/QC/Nanopore.fq.gz -P QC

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/QC/454.fq.gz -P QC

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/QC/PacBio.fq.gz -P QC

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/QC/QC.md5 -P QC
```

MiSeq data (soil fungi):
```bash
mkdir -p MiSeq_Fungi

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/MiSeq_Fungi/M{C,P}_1__R{1,2}.fastq.gz -P MiSeq_Fungi

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/MiSeq_Fungi/OC_{1..4}__R{1,2}.fastq.gz -P MiSeq_Fungi

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/MiSeq_Fungi/OP_{1..3}__R{1,2}.fastq.gz -P MiSeq_Fungi
```

Database for taxonomy annotation:
```bash
wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/DB/UNITE_Fungal_ITS.fa
```

Data for individual projects:
```bash
mkdir -p Set0{1..3}

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/Set01/Set01_s{1..5}_R{1,2}.fastq.gz -P Set01

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/Set02/Set02_s{1..5}_R{1,2}.fastq.gz -P Set02

wget https://raw.githubusercontent.com/Mycology-Microbiology-Center/Metabarcoding2022/main/data/Set03/Set03_s{1..5}_R{1,2}.fastq.gz -P Set03
```


## UNOISE pipeline

UNOISE is an algorithm for denoising (error-correcting) Illumina amplicon reads (Edgar, 2016; [DOI:10.1101/081257](https://www.biorxiv.org/content/10.1101/081257v1)). Initially, it was implemented in [USEARCH](https://drive5.com/usearch/) program, but now it's also available in [VSEARCH](https://github.com/torognes/vsearch) (Rognes et al., 2016; [DOI:10.7717/peerj.2584](https://peerj.com/articles/2584/)).

Start a new Docker container with VSEARCH (**NB!** remember to change the path to the directory with input data):
```bash
docker run -it -v /c/Metabarcoding:/data pipecraft/vsearch:2.18
cd /data
```

As detecting sequencing and PCR errors require information on the read abundances, we must dereplicate the data first. We will use chimera-filtered reads from the previous results.
```bash
vsearch \
  --derep_fulllength chimera_Filtered_out/MC_1_.fasta \
  --output MC_1_dereplicated.fasta \
  --fasta_width 0 \
  --relabel_sha1  --sizeout
```

Run UNOISE for a single sample:

```bash
vsearch \
  --cluster_unoise MC_1_dereplicated.fasta \
  --strand both \
  --minsize 4 \
  --unoise_alpha 2 \
  --fasta_width 0 \
  --sizein --sizeout \
  --centroids MC_1_unoise.fasta
```

Count how many zOTUs (zero-radius OTUs) were detected in the denoized sample and how many sequences were discarded:
```bash
grep -c "^>" MC_1_dereplicated.fasta  # input data
grep -c "^>" MC_1_unoise.fasta        # denoized data
```

To go through all samples, we can write a for-loop:
```bash
mkdir -p UNOISE

for i in chimera_Filtered_out/*.fasta; do

  vsearch \
    --derep_fulllength ${i} \
    --output - \
    --fasta_width 0 \
    --relabel_sha1  --sizeout \
  | vsearch \
    --cluster_unoise - \
    --strand both \
    --minsize 4 \
    --unoise_alpha 2 \
    --fasta_width 0 \
    --sizein --sizeout \
    --centroids UNOISE/$(basename ${i})

done
```

After UNOISE, you may proceed with OTU clustering using `PipeCraft2` GUI.
