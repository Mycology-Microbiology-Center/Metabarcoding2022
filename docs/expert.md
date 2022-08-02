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
  --minsize 8 \
  --fasta_width 0 \
  --sizein --sizeout \
  --centroids MC_1_unoise.fasta
```

Count how many zOTUs (zero-radius OTUs) were detected in the denoized sample and how many sequences were discarded:
```bash
grep "^>" chimera_Filtered_out/MC_1_.fasta | wc -l  # input data
grep "^>" MC_1_unoise.fasta                | wc -l  # denoized data
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
    --minsize 8 \
    --fasta_width 0 \
    --sizein --sizeout \
    --centroids UNOISE/$(basename ${i})

done
```

After UNOISE, you may proceed with OTU clustering using `PipeCraft2` GUI.



## HPC basics

For demonstration purposes, we will use the [Rocket Cluster](https://hpc.ut.ee/services/HPC-services/Rocket) of the University of Tartu.

To connect to the head-node of the HPC cluster using the secure shell protocol, run:
```bash
ssh USER@SERVER
```
substitute `USER` with your login ID and `SERVER` with a hostname (IP or a domain name).<br/>
E.g., `ssh koljalg@rocket.hpc.ut.ee`.

To copy a file or multiple files **to the HPC cluster**, use:
```bash
scp yourfile koljalg@rocket.hpc.ut.ee:~/yourfile   # single file
scp file1 file2 koljalg@rocket.hpc.ut.ee:~/        # multiple files
```
To copy file **from the HPC cluster** (e.g., `yourfile` from home directory on HPC to home directory on your computer), use:
```bash
scp koljalg@rocket.hpc.ut.ee:~/yourfile ~/yourfile
```

If you have large files (or a large number of files), it's better to use [`rsync`](https://en.wikipedia.org/wiki/Rsync) program for file transfer, e.g.
```bash
rsync -avz Documents/* koljalg@rocket.hpc.ut.ee:~/all/
```

To end your session on the HPC cluster, run:
```bash
exit
```

## Setup working environment on HPC cluster

In general, one needs admin rights to install the software on HPC clusters. However, users may install software into their home directory where they have write permissions. To make life easier, you may use `Conda` - a package manager which helps you find and install the software and its dependencies.<br/>
To install [Miniconda](https://docs.conda.io/en/latest/index.html), run the following code:
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p $HOME/miniconda
~/miniconda/bin/conda init bash
source ~/.bashrc
conda update --all --yes -c bioconda -c conda-forge
conda install --yes -c conda-forge mamba
```

To install the software (e.g., [`seqkit`](https://bioinf.shenwei.me/seqkit/) program), run:
```bash
mamba install -c bioconda seqkit
```


### Conda environments

If the software you wish to use could not be installed in the base (default) environment due to the conflict of versions, or you want to use a specific version of the program, or just want to keep it independent, you may create a separate environment with:
```bash
mamba create --name VSEARCHENV -c bioconda -c conda-forge vsearch=2.21.1 blast=2.13.0
conda activate VSEARCHENV           # swith to the new environment we've created
```

Verify which software versions are installed:
```bash
vsearch --version
blastn -version
```

To switch to the base environment, run:
```bash
conda deactivate
```

### Module system

Alternatively, if the software you wish to use is pre-installed on the HPC cluster, you may load it as an environment module.<br/>
To list all available modules, use `module avail` command (scroll the list with `space` button, press `q` to quit).<br/>
To search for a particular module, use e.g. `module -r spider '.*singularity.*'`.<br/>
If the required software was found, you need to load the module, e.g.:
```bash
module load any/singularity/3.7.3
```


## Scheduling jobs on the HPC cluster

The Slurm Workload Manager, a.k.a. Simple Linux Utility for Resource Management (SLURM), is used to share the HPC resources between users.

Please note that users log in from their computers to the cluster **head node**.<br/>
**Do not run the analysis on the head node!**<br/>
You should use it only to schedule the tasks that would be distributed on the **cluster nodes**.

To run the program on the HPC cluster you should:
- Prepare a batch script with directives to SLURM about the number of CPUs<sup>\*</sup>, amount of RAM<sup>\*</sup>, and time duration requested for the job, along with commands which perform the desired calculations;<br/>
- Submit the script to the batch queue. SLURM will evaluate the task's priority and start executing the job when it reaches the front of the queue.<br/>
When the job finishes, you may retrieve the output files.

<sup>\*</sup> Unless specified otherwise, on the Rocket cluster, all jobs will be allocated 1 node with 1 CPU core and 2 GB of memory.

### Batch script

Here is a basic batch script which that contains a minimal number of SLURM options:
```bash
#!/bin/bash -l
#SBATCH --job-name=my_job
#SBATCH --cpus-per-task=4
#SBATCH --nodes=1
#SBATCH --mem=10G
#SBATCH --partition amd
#SBATCH --time=48:00:00

## If needed, you may load the required modules here
# module load X

## Run your code
some_program -i input.data -o output_1.data --threads 4
some_script.sh output_1.data > output_2.data
echo "Done" > output.log
```

The syntax for the SLURM directive in a script is `#SBATCH <flag>`, where `<flag>` could be:<br/>
- `--job-name`, the name of the job;
- `--cpus-per-task`, the number of CPUs each task should have (e.g., 4 cores);
- `--nodes`, requested number of nodes (each node could have multiple CPUs);
- `--mem`, requested amount of RAM (e.g., 10 gigabytes);
- `--partition`, the partition on which the job shall run
- `--time`, the requested time for the job (e.g., 48 hours).

To submit a job, save the code above to a file (e.g., `my_job.sh`) and run:
```bash
sbatch my_job.sh
```

### Scheduling a task directly from the command line

If the command you wish to run is relatively simple, you may run it without a batch script, but in that case, you should provide SLURM directives as arguments to the `sbatch` command:
```bash
sbatch \
  --job-name=my_job \
  --ntasks-per-node=4 --nodes=1 --mem=10G -p amd \
  --time=48:00:00 \
  some_script.sh input.data
```

### Job management

When the job is submitted, you may monitor the queue and see the status of your running tasks:
```bash
squeue -u $USER
```
The most common job state codes (column `ST`) are:<br/>
`PD` =   PENDING<br/>
`R`  =   RUNNING<br/>
`S`  =   SUSPENDED<br/>

To cancel the job, use:
```bash
scancel <JOBID>        # by job ID (e.g., where <JOBID> is 31727880 - see the column JOBID in "squeue" output)
scancel --name my_job  # one or more jobs by name
scancel -u $USER       # all jobs for a current user
```
